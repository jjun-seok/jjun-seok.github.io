---
title: "웹 크롤링 대신 API를 쓰는 이유 - JSON/XML 수집과 멀티스레드 병렬 처리"
date: 2026-07-06 15:00:00 +0900
categories: [이어드림스쿨, 데이터수집]
tags: [api, crawling, json, xml, requests, threadpoolexecutor, python]
---

## 왜 크롤링 대신 API를 쓰는가

웹사이트를 직접 크롤링하려면 HTML 구조를 분석해야 하고, 사이트가 디자인을 바꿀 때마다 코드를 다시 고쳐야 한다. 게다가 대형 플랫폼은 크롤링 자체를 약관으로 막아두는 경우가 많아서 법적으로도 위험하다.

반면 **API**는 데이터를 제공하는 쪽이 "이 형식으로 요청하면 이 형식으로 데이터를 줄게"라고 미리 약속해둔 정식 창구다. 서울시 열린데이터광장 같은 공공데이터 포털이 대표적이다. 이번 글은 이 API 방식으로 데이터를 수집하는 전체 흐름 — JSON/XML 요청, 병렬 수집까지 — 을 정리한다.

---

## 1. JSON API로 데이터 요청하기

### 왜 인증키가 필요한가

공공데이터포털 API는 누구나 무료로 쓸 수 있지만, 남용을 막기 위해 **인증키(SERVICE_KEY)**로 요청자를 구분한다. 이 키를 URL에 끼워 넣어야 서버가 요청을 승인한다.

```python
import requests
import pandas as pd

# 서울시 열린데이터광장에서 발급받은 인증키
SERVICE_KEY = 'YOUR_SERVICE_KEY'  # 실제 발급받은 키로 교체

# 요청 규칙: [인증키]/[출력형식]/[서비스명]/[시작번호]/[끝번호]/[계약년도]
url = f'http://openapi.seoul.go.kr:8088/{SERVICE_KEY}/json/tbLnOpendataRtmsV/1/1000/2025'

req = requests.get(url)
content = req.json()                              # JSON 텍스트 → 파이썬 딕셔너리로 변환
con = content['tbLnOpendataRtmsV']['row']          # 실제 거래 데이터 배열만 추출
result = pd.DataFrame(con)                          # 표 형태로 변환

result.shape   # (1000, 컬럼수) 정도로 나오면 정상 수집
```

> API 응답 URL은 대부분 `[출력형식]/[서비스명]/[시작번호]/[끝번호]/[기간]` 순서로 규칙이 정해져 있는데, 이 규칙은 기관마다 완전히 다르다. 반드시 해당 API의 공식 문서를 먼저 확인해야 한다.
{: .prompt-tip }

### 🔴 오류 코드 vs 정상 코드 — 인증키를 코드에 그대로 박아두면?

```python
# ❌ 위험한 방식: 인증키를 코드에 하드코딩해서 GitHub에 그대로 올림
SERVICE_KEY = '586d67554b67737738306450656b4c'   # 실제 키가 공개 저장소에 노출됨
```

이렇게 올리면 인증키가 공개 저장소에 그대로 노출돼서 누구나 가져다 쓸 수 있고, 사용량 제한에 걸리거나 서비스가 정지될 수 있다.

```python
# ✅ 안전한 방식: 환경변수나 .env 파일로 분리
import os
from dotenv import load_dotenv

load_dotenv()
SERVICE_KEY = os.environ['SEOUL_API_KEY']   # .env 파일이나 OS 환경변수에서 읽어옴
```

`.env` 파일은 `.gitignore`에 등록해서 절대 커밋되지 않게 해야 한다.

---

## 2. XML API로 데이터 요청하기 — JSON과 뭐가 다른가

### JSON vs XML

- **JSON**: `{}`와 `[]` 기반의 속성-값 쌍. 가볍고 파이썬 딕셔너리로 바로 변환됨.
- **XML**: `<row>`, `<SGG_NM>`처럼 사용자가 직접 정의한 태그로 계층 구조를 표현. 공공기관 API에서 여전히 표준으로 많이 쓰인다.

같은 서울시 API라도 URL의 `json` 자리를 `xml`로만 바꾸면 XML 형식으로 응답을 받을 수 있다.

```python
import requests
import xml.etree.ElementTree as ET
import pandas as pd

SERVICE_KEY = 'YOUR_SERVICE_KEY'
url = f'http://openapi.seoul.go.kr:8088/{SERVICE_KEY}/xml/tbLnOpendataRtmsV/1/1000/2025'

resp = requests.get(url)
print("Status Code:", resp.status_code)   # 200이면 성공, 403이면 인증 실패, 404면 주소 오류

if resp.status_code == 200:
    xml_text = resp.text.strip()
    root = ET.fromstring(xml_text)          # 텍스트를 트리 구조(Element 객체)로 파싱

    rows = []
    for row in root.findall("row"):          # <row> 태그를 하나씩 순회
        row_dict = {}
        for child in row:                    # <row> 안의 자식 태그들
            row_dict[child.tag] = child.text  # 태그 이름=key, 태그 내용=value
        rows.append(row_dict)

    df = pd.DataFrame(rows)
    print("DataFrame shape:", df.shape)
    print(df.head())
else:
    print("HTTP 에러:", resp.text[:300])
```

### XML 파싱 흐름 한눈에 보기

`ET.fromstring(xml_text)`로 XML 문자열을 트리 구조로 바꾸면, `root.findall("row")`로 원하는 태그를 반복적으로 찾아 순회할 수 있다. 각 `<row>` 안의 자식 태그(`child.tag`, `child.text`)를 딕셔너리 키-값으로 매핑해서 리스트에 쌓으면, 그대로 `pd.DataFrame()`에 넣어 표로 만들 수 있다.

---

## 3. 멀티스레드로 전체 데이터 수집하기

### 왜 필요한가

공공데이터 API는 보통 한 번 요청에 1,000건 제한이 걸려있다. 2025년 전체 데이터가 5만 건이라면 요청을 50번 나눠서 보내야 하는데, 하나씩 순서대로 기다리면 시간이 오래 걸린다. **ThreadPoolExecutor**로 여러 요청을 동시에 보내면 속도가 훨씬 빨라진다(수업 자료 기준 약 10배).

```python
import time
import requests
import pandas as pd
from concurrent.futures import ThreadPoolExecutor, as_completed

SERVICE_KEY = 'YOUR_SERVICE_KEY'
SERVICE_NAME = 'tbLnOpendataRtmsV'
YEAR = '2025'
HEADERS = {"User-Agent": "Mozilla/5.0 ..."}   # 봇 차단 방지용 헤더

# 1단계: 전체 데이터 개수 먼저 조회
check_url = f'http://openapi.seoul.go.kr:8088/{SERVICE_KEY}/json/{SERVICE_NAME}/1/1/{YEAR}'
res = requests.get(check_url, headers=HEADERS)
meta_data = res.json()
total_count = meta_data[SERVICE_NAME]['list_total_count']

# 2단계: 1,000건 단위로 쪼갠 URL 목록 생성
api_urls = []
for start in range(1, total_count + 1, 1000):
    end = min(start + 999, total_count)
    api_urls.append(f'http://openapi.seoul.go.kr:8088/{SERVICE_KEY}/json/{SERVICE_NAME}/{start}/{end}/{YEAR}')

# 3단계: 한 구간을 가져오는 함수 정의
def fetch_api_chunk(url):
    try:
        req = requests.get(url, headers=HEADERS, timeout=10)
        if req.status_code != 200:
            return None
        return req.json()[SERVICE_NAME]['row']
    except Exception:
        return None

# 4단계: 스레드 풀에 동시에 던지고 끝나는 대로 결과 취합
all_rows = []
MAX_WORKERS = 5   # 서버 과부하 방지를 위해 5~8개가 안전
with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
    futures = {executor.submit(fetch_api_chunk, url): url for url in api_urls}
    for future in as_completed(futures):
        chunk_data = future.result()
        if chunk_data:
            all_rows.extend(chunk_data)

df_all = pd.DataFrame(all_rows)
df_all.to_csv("seoul_real_estate_2025_master.csv", index=False, encoding="utf-8-sig")
```

### 동작 원리 — 왜 빨라지나

순서대로 요청하면 "1번 요청 응답 기다림 → 2번 요청 응답 기다림 → ..."처럼 대기 시간이 그대로 누적된다. **ThreadPoolExecutor**는 이 "응답 기다리는 시간"에 다른 요청들을 동시에 던져놓기 때문에, 전체 대기 시간이 겹쳐지면서 총 소요 시간이 확 줄어든다.

> `MAX_WORKERS`를 너무 높게 잡으면 서버에 부담을 줘서 오히려 요청이 차단될 수 있다. 공공데이터 API는 보통 5~8개 정도가 안전하다.
{: .prompt-warning }

진행 상황을 눈으로 확인하고 싶으면 `tqdm`을 `as_completed`와 같이 쓰면 진행률 바가 실시간으로 표시된다.

```python
from tqdm import tqdm

for future in tqdm(as_completed(futures), total=len(api_urls), desc="수집률"):
    ...
```

---

## 크롤링 vs API 요약 비교

| | 직접 크롤링 | API 활용 |
|---|---|---|
| 필요 지식 | HTML 문법 이해 필수 | 요청 URL 규칙 이해 |
| 안정성 | 사이트 구조 바뀌면 코드 깨짐 | 공식 문서 기준이라 안정적 |
| 합법성 | 약관 위반 소지 있음 | 정식 허용된 방식 |
| 속도 향상 | 멀티스레드 적용 가능 | 멀티스레드 적용 가능 (더 안전하게) |
| 한계 | 없음 (사이트가 막지 않는 한) | 무료 한도 있음, 필요시 유료 결제 |

---

## 오늘의 핵심 정리

1. 크롤링은 HTML을 직접 해석해야 하고 사이트 변경에 취약하다 — 가능하면 API를 우선 고려한다.
2. JSON은 파이썬 딕셔너리로 바로 변환되고, XML은 `ElementTree`로 트리 파싱 후 태그별로 딕셔너리를 만들어야 한다.
3. API 인증키는 절대 코드에 하드코딩해서 공개 저장소에 올리면 안 되고, 환경변수나 `.env`로 분리해야 한다.
4. 요청 건수 제한이 있는 API는 `ThreadPoolExecutor`로 병렬 요청을 보내면 수집 속도를 크게 줄일 수 있다 — 단, 서버 부담을 고려해 워커 수는 적당히(5~8개) 제한한다.

---

## 참고

- [서울 열린데이터광장 - 부동산 실거래가 정보](https://data.seoul.go.kr/dataList/OA-15548/S/1/datasetView.do)
- [requests 공식 문서](https://requests.readthedocs.io/en/latest/)
- [Python - xml.etree.ElementTree 공식 문서](https://docs.python.org/3/library/xml.etree.elementtree.html)
- [Python - concurrent.futures 공식 문서](https://docs.python.org/3/library/concurrent.futures.html)
