# FRS for Transaction Hub
## Version
| Date       | Version | Author | Description     |
|------------|---------|--------|-----------------|
| 18-10-2023 | 1.0     | DatDT  | Initial Version |
## Abbreviation
- Physical Terminal: PT
- Virtual Terminal: VT
- Working Key: WK
- Serial Number: SN
- Average time of success transaction: ATT
- Min time delay between transactions: MinTDL
- Max time delay between transactions: MaxTDL
- Timeout for request send from physical terminal to virtual terminal: PTTO
- Timeout for request send from virtual terminal to processor: VTTO
## I. Overview
- Language: Golang
- Database: MS-SQL
- Total estimation time: 154h
## II. Transaction Hub APIs
- APIs for PT (Post API.docx)
  - Get working key
  - Request payment
  - Check transaction
  - Balance Inquiry
- APIs for web
  - Add VT and configuration for VT:
    - Parameters:
      - VT ID
      - Processor ID
      - Max PT
      - Cassette Info
      - Low Amount
      - Working key exchange interval
      - Protocol Type
      - Delay time between transactions
      - HealCheck interval
      - Timeout for request send to processor (VTTO)
      - Working key authorization (Fixed value)
      - Min time delay between transactions (MinTDL)
      - Max time delay between transactions (MaxTDL)
  - Add PT and configuration for PT:
    - Parameters:
      - PT ID
      - VT ID
      - PT Serial Number
      - Timeout for request send to VT (PTTO)
  - Add Processor:
    - Parameters:
      - Name
      - Server IP
      - Server Port
      - Enable Staging
      - Enable TLS
      - Timeout for request from VT
      - Supported protocol
      - Average time of success transaction (ATT)
## III. Transaction Hub Modules
### 1. API
#### 1.1. Get working key
- Draft flow: PT SN => PT ID => VT ID => Get VT WK from DB
  - if WK == NULL: Call processor to get WK for this VT => Update WK to DB => return WK
  - else: return WK
- Estimate:
  - Implement: 10h
  - Unit test: 4h
  - Total: 14h
#### 1.2. Request payment
- Draft flow: 
  - Provider thread: Push request to queue => Wait for reponse
  - Consumer thread: Receive message => Get VT ID from PT SN => Check number pending message of VT (n) => Calculate time to wait (ATT*n + MaxTDL*(n-1)) to accept or reject request 
    - If time to wait > (PTTO - ATT): Send reject response to provider thread
    - else: When it picks message from queue:
      - Calculate time need to delay (TDL): Random(MinTDL, MaxTDL) - (now() - time(last VT TXN))
      - Check (message received time + PTTO) < now() + TDL + ATT:
        - False: Send response reject process request because not enough time
        - True: Process request => Convert PT request to VT request => Receive response from processor => Convert to PT response => Return to provider
- Estimate:
  - Research for provider and consumer mechanism: 7h
  - Implement provider logic: 7h
  - Implement consumer logic: 14h
  - Unit test: 7h
  - Total: 35h
#### 1.3. Transaction check
- Draft flow: Parse request => Query DB to check TXN info and TXN status => Return response
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 1.4. Balance Inquiry
- Draft flow: Parse request => Same logic to payment request (Call to processor and change some fields) => Return response
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 1.5. Add VT
- Draft flow: Parse request => Update DB => Return response
- Estimate:
  - Implement: 4h
  - Unit test: 2h
  - Total: 6h
#### 1.6. Add PT
- Draft flow: Parse request => Update DB => Return response
- Estimate:
  - Implement: 4h
  - Unit test: 2h
  - Total: 6h
### 2. Schedule Jobs
#### 2.1. Get working key (interval=VT.Working key exchange interval)
- Draft flow: Send request to processor to get WK for VT => Update WK to DB
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 2.2. Cash Position (interval=VT.HealCheck interval)
- Draft flow: Check VT balance < VT.LowAmount => Send request to processor to do Cash Position (random [1000, 1500, 1800])
=> Check response => Success => Update VT Cassette infomation
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
### 3. Database package
- Create package to query, update VT,PT,Transaction... to MSSQL
- Create interface for main package to call
- Estimate:
  - Implement: 14h
  - Unit test: 7h
  - Total: 21h
### 4. Processor communication package
- Create package to map and parse processor data (json <=> binary)
- Create interface for main package to call
- Estimate:
  - Research (read processor document and code example): 7h
  - Implement: 14h
  - Unit test: 7h
  - Total: 28h



