# Smart Freight Sharing Platform (B2B/C2C Logistics Marketplace)

Business Analysis & Product Specification Portfolio  
Author: Business Analyst (Fresher++ / Junior)  
Domain: Supply Chain / Domestic Road Freight Transportation (Less-Than-Truckload - LTL)  
Interactive Prototype Demo: [Live Web Prototype](prototype_freight_sharing.html)

---

## 1. Executive Summary & Problem Statement (BACCM)

- **Context:** In domestic inter-provincial road freight transport, long-haul trucks frequently experience 50% to 70% empty return trips (deadhead miles) after initial delivery, wasting fuel and operational costs.
- **Core Problem:** Small/LTL shippers pay high fees for full truck rentals or slow postal consolidation, while return-trip truck drivers lack a reliable platform to find cargo along their return corridor.
- **Solution:** A two-sided freight matching marketplace integrating Multi-factor Smart Matching, Anti-Fraud Electronic Proof of Delivery (e-PoD), and Fintech Escrow Payment Custody with a 24-hour Auto-Disbursement Engine.

---

## 2. System Context Diagram (DFD Level 0)

```mermaid
flowchart TD
    SYS["SAN DIEU PHOI XE TAI GHEP HANG<br/>(Smart Freight Sharing Platform)"]
    DRV["Driver<br/>(Tai xe)"]
    SHP["Shipper<br/>(Chu hang)"]
    CSG["Consignee<br/>(Nguoi nhan)"]
    ADM["Admin<br/>(Quan tri vien)"]
    PAY["Payment Gateway<br/>(VietQR / VNPAY)"]
    MAP["Map & Telematics API<br/>(Vietmap / Google)"]

    DRV -->|"Dang chuyen ve, Nhan don,<br/>Gui e-PoD, Rut tien vi"| SYS
    SYS -->|"Voice Alert, Don ghep phu hop,<br/>Giai ngan 90% tien cuoc"| DRV

    SHP -->|"Tao don hang, Nap coc Escrow,<br/>Gui khieu nai (Dispute)"| SYS
    SYS -->|"Goi y danh sach xe tai,<br/>Live GPS, Bien ban e-PoD"| SHP

    CSG -->|"Ky cam ung e-PoD,<br/>Cung cap ma SMS OTP"| SYS
    SYS -->|"Gui ma SMS OTP xac thuc,<br/>Thong bao trang thai giao"| CSG

    ADM -->|"Kiem duyet KYC tai xe,<br/>Phan quyet tranh chap"| SYS
    SYS -->|"Bao cao doi soat doanh thu,<br/>Danh sach ticket khieu nai"| ADM

    SYS -->|"Yeu cau thanh toan coc"| PAY
    PAY -->|"Webhook xac nhan thanh toan"| SYS

    SYS -->|"Yeu cau dinh tuyen ban do"| MAP
    MAP -->|"Vet Polyline, Canh bao cam tai"| SYS
```

---

## 3. End-to-End Business Workflow

```mermaid
flowchart TD
    A["1. Tai xe dang chuyen ve<br/>& Chu hang tao don hang"] --> B["2. Thuat toan quet Matching:<br/>Check W, V, Lech tuyen, Loai duong"]
    B --> C{"Co xe<br/>phu hop?"}
    C -- Co --> D["3. Chu hang chon xe phu hop<br/>& Khoa 100% coc vao Quy Escrow"]
    C -- Khong --> E["Dang don len san cho PENDING<br/>de cho tai xe khac nhan"]
    D --> F["4. Tai xe den lay hang,<br/>chup anh doi chung (IN_TRANSIT)"]
    F --> G["5. Den noi: Check GPS <=200m,<br/>Chup 2 anh & Ky e-PoD (DELIVERED)"]
    G --> H["6. Kich hoat bo dem nguoc<br/>24 gio Escrow tu dong"]
    H --> I{"Trong 24h<br/>co Khieu nai?"}
    I -- KHONG co khieu nai --> J["Tu dong tru 10% phi san,<br/>Giai ngan 90% cho Tai xe (COMPLETED)"]
    I -- CO khieu nai --> K["Dong bang tien Escrow (FROZEN)<br/>& Admin vao doi soat phan xu"]
```

---

## 4. Detailed Core Use Cases & Business Rules

### UC-01: Smart Matching Engine & Escrow Lock
- **Matching Criteria:** Checks payload weight (kg), volumetric weight (CBM), and route corridor detour within 10 km buffer zone.
- **Road Accessibility Filter (FN-02.1B):** Automatically filters out Container Trucks (> 5 tons) if the Shipper indicates a narrow alley destination (<= 2.5 tons only), providing an alternative "Roadside Gas Station Drop-off" option.
- **Escrow Pre-authorization:** 100% of freight fee is locked upon match confirmation via Dynamic VietQR / Bank Card gateway.

### UC-02: Electronic Proof of Delivery (e-PoD)
- **3-Factor Verification:**
  1. **GPS Geofencing:** Validates driver device distance is <= 200 meters from destination. Includes Anti-Mock Location check to prevent fake GPS apps.
  2. **Live Camera Capture:** Enforces live camera capture (>= 2 photos) and disables gallery upload.
  3. **Digital Signature:** Touchscreen finger signature with SMS OTP fallback if recipient is absent.
- **Offline-First Sync:** Stores encrypted proof locally during network outages and auto-syncs when 4G reconnects.

### UC-03 & UC-04: 24-Hour Escrow Settlement & Dispute Arbitration
- **Settlement Formula:**
  - Driver Net Payout = 90% of Total Freight Fee
  - Platform Commission Fee = 10% of Total Freight Fee
- **Dispute Freezing:** Instant freeze of the 24-hour countdown upon complaint submission. Admin audits baseline pick-up photos vs. drop-off e-PoD photos to rule refund or payout within a 48-hour SLA.

---

## 5. Data Model Entities (High-Level ERD)

1. `Users`: user_id, phone_number, full_name, role (SHIPPER, DRIVER, ADMIN), wallet_balance.
2. `Vehicles`: vehicle_id, driver_id, plate_number, truck_type, max_weight_kg, max_cbm, dimensions_LxWxH.
3. `Trips`: trip_id, driver_id, origin, destination, route_polyline, etd, available_weight_kg, available_cbm, trip_status.
4. `Orders`: order_id, shipper_id, pickup_address, dropoff_address, weight_kg, cbm, road_access_type, shipping_fee, order_status.
5. `ePoD_Records`: epod_id, order_id, delivery_gps_lat, delivery_gps_long, proof_photo_urls, signature_image_url, delivered_at.
6. `Escrow_Transactions`: escrow_id, order_id, total_amount, commission_fee, driver_payout_amount, escrow_status, auto_disburse_at.

---

## 6. Repository Artifacts

- `Context_Diagram.drawio`: Complete System Context DFD Level 0 diagram.
- `Workflow_Diagram.drawio`: End-to-End multi-phase Business Workflow diagram.
- `Activity_Diagrams.drawio`: Multi-page Swimlane Activity Diagrams for Matching, e-PoD, and Escrow.
- `prototype_freight_sharing.html`: Standalone interactive prototype demo (HTML5, Vanilla CSS, JS).
