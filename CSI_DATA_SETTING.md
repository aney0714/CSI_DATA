# ESP32-S3 CSI DATA 수집 과정 정리

# 1. ESP-IDF 5.4 환경 활성화

PowerShell 실행 후:

```powershell
cd C:\Users\gimhh\esp\v5.4.4\esp-idf
.\export.ps1
```

정상 실행 시:

```txt
Done! You can now compile ESP-IDF projects.
```

출력 확인.

---

# 2. ESP32-CSI-Tool 프로젝트 이동

```powershell
cd C:\esp\ESP32-CSI-Tool
```

프로젝트 구조:

```txt
active_ap   → RX(AP)
active_sta  → TX(STA)
```

---

# 3. RX(active_ap) 설정 및 빌드

RX용 ESP32-S3 보드 연결 후(COM4 기준):

```powershell
cd C:\esp\ESP32-CSI-Tool\active_ap
```

타겟 설정:

```powershell
idf.py set-target esp32s3
```

menuconfig 실행:

```powershell
idf.py menuconfig
```

---

# 4. menuconfig에서 CSI 활성화

다음 항목 활성화:

```txt
ESP32 CSI Tool Config
→ [*] Should this ESP32 collect and print CSI data?
→ [*] Send CSI data to Serial
```

추가로:

```txt
Component config
→ Wi-Fi
→ WiFi CSI(Channel State Information)
```

활성화 필요.

검색 방법:

```txt
/ 키 → CSI 검색
```

저장 후 종료:

```txt
S 저장
Q 종료
```

---

# 5. RX(active_ap) Build 및 Flash

```powershell
idf.py fullclean
idf.py build
idf.py -p COM4 -b 115200 flash
```

Flash 완료 후 monitor 실행:

```powershell
idf.py -p COM4 monitor
```

정상 실행 시:

```txt
Active CSI collection (AP)
```

출력 확인.

---

# 6. TX(active_sta) 설정 및 빌드

새 터미널 실행 후:

```powershell
cd C:\Users\gimhh\esp\v5.4.4\esp-idf
.\export.ps1

cd C:\esp\ESP32-CSI-Tool\active_sta
```

TX task crash 해결을 위해:

```txt
main/main.cc
```

파일 수정.

기존 코드:

```cpp
xTaskCreatePinnedToCore(..., 10000, (void *)&is_wifi_connected, 100, ...);
```

수정 코드:

```cpp
xTaskCreatePinnedToCore(..., 8192, NULL, 5, ...);
```

---

# 7. TX(active_sta) Build 및 Flash

```powershell
idf.py fullclean
idf.py set-target esp32s3
idf.py build
idf.py -p COM6 -b 115200 flash
```

monitor 실행:

```powershell
idf.py -p COM6 monitor
```

정상 실행 시:

```txt
Active CSI collection (Station): connect to ap SSID:myssid
```

출력 확인.

---

# 8. CSI_DATA 출력 성공

RX(active_ap) monitor에서 아래 형태의 데이터 출력 확인:

```txt
CSI_DATA,AP,64:49:7D:EB:42:40,-27,...
```

현재 확인된 내용:

* AP ↔ STA WiFi 연결 성공
* WiFi packet 송수신 성공
* CSI callback 정상 동작
* Raw CSI Data Serial 출력 성공

---

# 9. CSV 저장 방법

RX monitor에서 CSV 저장:

```powershell
idf.py -p COM4 monitor | findstr "CSI_DATA" > test_01.csv
```

설명:

```txt
monitor                 → Serial 출력 읽기
findstr "CSI_DATA"      → CSI_DATA 줄만 추출
> test_01.csv           → CSV 파일 저장
```

---

# 10. 현재 프로젝트 상태

현재까지 완료된 흐름:

```txt
ESP32-S3 AP(active_ap) 생성
→ ESP32-S3 STA(active_sta) 연결
→ WiFi packet 송수신
→ CSI callback 실행
→ CSI Raw Data 추출
→ Serial Monitor 출력 성공
→ CSV 저장 가능 상태 도달
```

다음 단계 예정:

```txt
CSV 저장
→ Python 데이터 수집
→ amplitude 계산
→ sliding window
→ filtering
→ AI 모델 입력
→ 낙상 감지
```
