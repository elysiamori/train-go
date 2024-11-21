# Train Go (Tiket Kereta API)

## 1. Product Requirement Document (PRD)

### Objective

Membangun sistem API tiket kereta yang mendukung:

- Pencarian jadwal kereta berdasarkan berbagai kriteria (stasiun, tanggal, waktu).
- Sistem reservasi yang mengelola tiket dan ketersediaan kursi.
- Validasi tiket dengan notifikasi pengguna sebelum keberangkatan.
- Monitoring penggunaan tiket secara real-time.

### Main Feature

1. **Manajemen Jadwal Kereta**:

   - Tambah, ubah, dan hapus jadwal kereta.
   - Informasi keberangkatan dan kedatangan.

2. **Manajemen Tiket**:

   - Pemesanan tiket.
   - Pembatalan tiket.
   - Validasi tiket.

3. **Pengingat Waktu**:
   - Notifikasi 15 menit sebelum keberangkatan.

### Role

- **User**: Penumpang kereta.
- **Admin**: Mengelola jadwal kereta dan tiket.

### Kebutuhan Teknis

- **Backend**: ``Golang``
- **Database**: ``PostgreSQL`` (untuk penyimpanan data utama)
- **Caching**: ``Redis`` (untuk caching dan antrean notifikasi)
- **Hosting**: ``Fly.io``
- **Notification**: ``Twilio``

---

## 2. Architecture Databases

### 1. Users

- **id (Primary Key)**: ID unik pengguna.
- **name**: Nama pengguna.
- **email**: Email pengguna.
- **password**: Password pengguna (hashed).
- **role**: Peran pengguna (user atau admin).

#### Relationship:

- **One-to-Many** ke **Tickets**.
- **One-to-Many** ke **Notifications**.

---

### 2. Trains

- **id (Primary Key)**: ID unik kereta.
- **name**: Nama kereta (contoh: Argo Bromo).
- **start_station**: Stasiun keberangkatan.
- **end_station**: Stasiun tujuan.
- **departure_time**: Waktu keberangkatan.
- **arrival_time**: Waktu kedatangan.

#### Relationship:

- **One-to-Many** ke **Seats**.
- **One-to-Many** ke **Tickets**.

---

### 3. Seats

- **id (Primary Key)**: ID unik untuk setiap kursi.
- **train_id (Foreign Key)**: Referensi ke kereta tertentu.
- **seat_number**: Nomor kursi (contoh: A1, B2).
- **is_booked**: Status apakah kursi sudah dipesan (true atau false).

#### Relationship:

- **Many-to-One** ke **Trains**.

---

### 4. Tickets

- **id (Primary Key)**: ID unik tiket.
- **user_id (Foreign Key)**: Referensi ke pengguna yang memesan tiket.
- **train_id (Foreign Key)**: Referensi ke kereta yang dipilih.
- **seat_id (Foreign Key)**: Referensi ke kursi yang dipesan.
- **status**: Status tiket (active, expired, cancelled).
- **booked_at**: Waktu tiket dipesan.

#### Relationship:

- **Many-to-One** ke **Users**.
- **Many-to-One** ke **Trains**.
- **One-to-Many** ke **Notifications**.

---

### 5. Notifications

- **id (Primary Key)**: ID unik notifikasi.
- **user_id (Foreign Key)**: Referensi ke pengguna yang menerima notifikasi.
- **ticket_id (Foreign Key)**: Referensi ke tiket terkait.
- **type**: Jenis notifikasi (reminder atau expired).
- **sent_at**: Waktu notifikasi dikirim.

#### Relationship:

- **Many-to-One** ke **Users**.
- **Many-to-One** ke **Tickets**.

## Relationship Antar Entitas

### 1. **Users → Tickets**

- **One-to-Many**: Satu pengguna dapat memiliki banyak tiket.
- **Relationship**: `Users.id → Tickets.user_id`

### 2. **Trains → Tickets**

- **One-to-Many**: Satu kereta dapat memiliki banyak tiket.
- **Relationship**: `Trains.id → Tickets.train_id`

### 3. **Trains → Seats**

- **One-to-Many**: Satu kereta dapat memiliki banyak kursi.
- **Relationship**: `Trains.id → Seats.train_id`

### 4. **Tickets → Notifications**

- **One-to-Many**: Satu tiket dapat memiliki banyak notifikasi.
- **Relationship**: `Tickets.id → Notifications.ticket_id`

### 5. **Users → Notifications**

- **One-to-Many**: Satu pengguna dapat menerima banyak notifikasi.
- **Relationship**: `Users.id → Notifications.user_id`

## 3. Flow

### Use Case Flow

1. **Pemesanan Tiket**:

   - User memilih jadwal kereta.
   - API memvalidasi ketersediaan kursi.
   - Tiket dibuat dengan status "active".

2. **Validasi Tiket**:
   - User memindai tiket sebelum keberangkatan.
   - API mengecek waktu saat ini dengan waktu keberangkatan.
   - Jika waktu saat ini melewati waktu keberangkatan, status tiket menjadi "expired".

---

## 4. API Documentation

### Base URL

`https://tiket-kereta.fly.dev`

### Endpoints

#### 1. Get Jadwal Kereta (Dengan Filter)

- **Method**: GET
- **Endpoint**: `/trains?start_station=Jakarta&end_station=Surabaya&date=2024-11-21`
- **Response**:

```json
[
  {
    "id": 1,
    "name": "Argo Bromo",
    "route": "Jakarta - Surabaya",
    "departure_time": "2024-11-21T08:00:00Z",
    "arrival_time": "2024-11-21T16:00:00Z"
  }
]
```

#### 2. Pesan Tiket (Dengan Kursi)

- **Method**: POST
- **Endpoint**: `/tickets`
- **Body**:

```json
{
  "train_id": 1,
  "seat_number": "A1"
}
```

- **Response**:

```json
{
  "ticket_id": 123,
  "status": "active"
}
```

#### 3. Validasi Tiket

- **Method**: POST
- **Endpoint**: `/tickets/validate`
- **Body**:

```json
{
  "ticket_id": 123
}
```

- **Response**:

```json
{
  "status": "valid"
}
```

#### 4. Statistik Penjualan Tiket (Admin)

- **Method**: GET
- **Endpoint**: `/admin/stats`
- **Response**:

```json
{
  "total_tickets": 200,
  "total_revenue": 50000000,
  "trains_on_time": 90,
  "trains_delayed": 10
}
```
