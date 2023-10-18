# FRS for Transaction Hub
<!-- TOC -->
* [FRS for Transaction Hub](#frs-for-transaction-hub)
  * [Version](#version)
  * [Abbreviation](#abbreviation)
  * [I. Technology](#i-technology)
  * [II. Hub APIs](#ii-hub-apis)
  * [III. Hub Logic](#iii-hub-logic)
    * [1. API](#1-api)
      * [1.1. Get working key](#11-get-working-key)
      * [1.2. Request payment](#12-request-payment)
      * [1.3. Transaction check](#13-transaction-check)
      * [1.4. Balance Inquiry](#14-balance-inquiry)
      * [1.5. Add VT](#15-add-vt)
      * [1.6. Add PT](#16-add-pt)
    * [2. Schedule Jobs](#2-schedule-jobs)
      * [2.1. Get working key (interval=VT.Working key exchange interval)](#21-get-working-key--intervalvtworking-key-exchange-interval-)
      * [2.2. Cash Position (interval=VT.HealCheck interval)](#22-cash-position--intervalvthealcheck-interval-)
    * [3. Database package](#3-database-package)
    * [4. Processor communication package](#4-processor-communication-package)
<!-- TOC -->
## Version
| Date       | Version | Author | Description     |
|------------|---------|--------|-----------------|
| 18-10-2023 | 1.0     | DatDT  | Initial Version |
## Abbreviation
- Physical Terminal: PT
- Virtual Terminal: VT
- Working Key: WK
- Serial Number: SN
## I. Technology
- Language: Golang
- Database: MS-SQL
## II. Hub APIs
- APIs for PT
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
      - Timeout for request from PT
      - Timeout for request send to processor
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
## III. Hub Logic
### 1. API
#### 1.1. Get working key
- Flow: PT SN => PT ID => VT ID => Get VT WK from DB
  - if WK == NULL: Call processor to get WK for this VT => Update WK to DB => return WK
  - else: return WK
- Estimate:
  - Implement: 10h
  - Unit test: 4h
  - Total: 14h
#### 1.2. Request payment
- Flow: 
  - Provider thread: Push request to queue => Wait for reponse
  - Consumer thread: Receive message => Get VT ID from PT SN => Check number pending message of VT => Calculate time to accept or reject request 
    - If rejected: Send reject response to provider thread
    - If accepted: When it picks message from queue:
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
- Flow: Parse request => Query DB to check TXN info and TXN status => Return response
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 1.4. Balance Inquiry
- Flow: Parse request => Same logic to payment request (Call to processor and change some fields) => Return response
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 1.5. Add VT
- Flow: Parse request => Update DB => Return response
- Estimate:
  - Implement: 4h
  - Unit test: 2h
  - Total: 6h
#### 1.6. Add PT
- Flow: Parse request => Update DB => Return response
- Estimate:
  - Implement: 4h
  - Unit test: 2h
  - Total: 6h
### 2. Schedule Jobs
#### 2.1. Get working key (interval=VT.Working key exchange interval)
- Flow: Send request to processor to get WK for VT => Update WK to DB
- Estimate:
  - Implement: 7h
  - Unit test: 4h
  - Total: 11h
#### 2.2. Cash Position (interval=VT.HealCheck interval)
- Flow: Check VT balance < VT.LowAmount => Send request to processor to do Cash Position (random [1000, 1500, 1800])
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



