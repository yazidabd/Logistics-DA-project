# 🚚 Logistics Data Analysis Project

## 🧭 Ringkasan Proyek
Proyek ini bertujuan untuk menganalisis performa operasional logistik perusahaan menggunakan pendekatan **data-driven**.  
Analisis dilakukan melalui dua tahap utama:

1. **Exploratory Data Analysis (EDA)** menggunakan Python untuk memahami pola dan kualitas data.  
2. **Pembuatan Dashboard Interaktif di Power BI** untuk menampilkan insight utama terkait efisiensi, biaya, dan performa pengiriman.

---

## 🎯 Tujuan Bisnis
1. Menilai performa operasional pengiriman barang (volume, ketepatan waktu, dan biaya).  
2. Mengidentifikasi faktor utama yang mempengaruhi biaya dan durasi pengiriman.  
3. Menemukan anomali atau ketidakefisienan pada carrier maupun warehouse.  
4. Menyediakan visualisasi interaktif yang membantu tim bisnis memantau KPI logistik secara real time.

---

## 🧮 Tahapan Analisis Data (EDA - Python)
Tahapan eksplorasi dilakukan di file `01_logistics_eda.ipynb` menggunakan **pandas**, **numpy**, **matplotlib**, dan **seaborn**.

### 1️⃣ Persiapan Data
- **Dataset:** https://www.kaggle.com/datasets/shahriarkabir/us-logistics-performance-dataset/
- Mengecek struktur (`shape`, `info`, `describe`) dan jumlah nilai unik tiap kolom.
- **Menangani missing value** dengan menghapus baris yang memiliki nilai kosong (`dropna`).

### 2️⃣ Deteksi dan Penghapusan Outlier
- Menggunakan metode **IQR (Interquartile Range)** untuk mendeteksi dan menghapus outlier pada kolom numerik:
  - `Weight_kg`
  - `Cost`
- Setelah outlier dihapus, dilakukan visualisasi **boxplot ulang** untuk memastikan distribusi sudah bersih.

### 3️⃣ Validasi Konsistensi Transit Days
- Mengonversi `Shipment_Date` dan `Delivery_Date` ke format datetime.
- Membuat kolom `Selisih_hari = Delivery_Date - Shipment_Date`.
- Membandingkan dengan kolom `Transit_Days`:
  - Jika beda > 1 hari → “Major Mismatch”
  - Jika beda ≤ 1 hari → “Minor Mismatch”
  - Jika sama → “OK”
- Menghapus baris dengan error besar atau tanggal negatif.

### 4️⃣ Analisis Eksploratif
- **Distribusi data:** histogram + KDE untuk `Weight_kg`, `Cost`, `Distance_miles`, dan `Transit_Days`.
- **Hubungan antar variabel:** scatterplot + regresi antara `Distance_miles` vs `Transit_Days`, dan `Weight_kg` vs `Cost`.
- **Perbandingan antar carrier:** rata-rata `Cost` dan `Transit_Days` per carrier menggunakan `groupby`.

> Dataset hasil akhir disimpan sebagai `logistics_clean.csv` (dataset utama untuk Power BI dashboard).

---

## 📊 Dashboard Power BI: Struktur & Insight

Dashboard dibagi menjadi **tiga halaman utama** untuk menjawab pertanyaan bisnis berbeda.

---

### 🟦 1. Logistics Performance Overview
**Tujuan:** Menilai performa operasional logistik secara umum.

#### Visual yang Dibuat:
- **KPI Cards** → Total Shipment, Average Cost, Average Transit Days, On-time Delivery Rate.  
  - *Measure contoh:*  
    ```DAX
    Total Shipment = COUNTROWS('logistics_clean')
    Avg Cost = AVERAGE('logistics_clean'[Cost])
    On-Time % = DIVIDE(
        COUNTROWS(FILTER('logistics_clean', 'logistics_clean'[Delivery_Status] = "Delivered")),
        COUNTROWS('logistics_clean'),
        0
    )
    ```
- **Bar Chart:** Distribusi pengiriman berdasarkan Carrier & Warehouse Asal.  
- **Map:** Lokasi pengiriman berdasarkan kota tujuan.  
- **Pie Chart:** Proporsi status pengiriman.

#### Insight Utama:
- **Performa pengiriman sangat baik** → On-time delivery rate **94,5%**.  
- Mayoritas pengiriman **(90,3%) berhasil delivered**, hanya **1,1% delay**.  
- Volume tinggi: **1.640 shipment**, dengan rata-rata **4 hari transit** dan biaya rata-rata **$194** per pengiriman.  
- **Distribusi demand merata** di kota besar (Denver, Portland, Chicago, Dallas, Phoenix).  
- **Warehouse SF & LA** paling produktif dengan >180 pengiriman.

---

### 🟩 2. Shipping Efficiency & Cost Analysis
**Tujuan:** Menganalisis efisiensi biaya dan faktor yang mempengaruhinya.

#### Visual yang Dibuat:
- **Scatter Plot:** Hubungan Cost vs Distance (berdasarkan Carrier).  
- **Line Chart:** Average Transit Days per Month (tren musiman).  
- **Table:** Cost per Mile by Carrier.  
  - *Measure contoh:*  
    ```DAX
    Cost per Mile = DIVIDE(SUM('logistics_clean'[Cost]), SUM('logistics_clean'[Distance_miles]), 0)
    ```
- **Slicer:** Filter Carrier, Warehouse, dan Bulan Pengiriman.

#### Insight Utama:
- **DHL memiliki cost per mile tertinggi** ($0,20), sementara **USPS paling efisien** ($0,16).  
- Tidak ada korelasi kuat antara berat dan jarak dengan biaya — **carrier & rute lebih dominan**.  
- Terdapat **pola musiman**: waktu pengiriman meningkat di **Mei (4,4 hari)** dan **November (4,6 hari)** (indikasi holiday season).  
- Efisiensi pengiriman: **4,2 hari per 1000 mil**, menunjukkan konsistensi waktu tempuh.

---

### 🟥 3. Deep Dive: Anomaly & Delay Analysis
**Tujuan:** Menemukan anomali biaya dan pola keterlambatan pengiriman.

#### Visual yang Dibuat:
- **Boxplot:** Distribusi biaya per carrier untuk mendeteksi outlier.  
- **Table Detail:** Menampilkan shipment dengan biaya ekstrem.  
- **Matrix Route Efficiency:** Menampilkan kombinasi Warehouse → Destination.  
  - *Measure contoh:*  
    ```DAX
    Avg Cost per Route = AVERAGE('logistics_clean'[Cost])
    Avg Transit Days per Route = AVERAGE('logistics_clean'[Transit_Days])
    ```

#### Insight Utama:
- **DHL** memiliki median cost tertinggi ($228), menjadi kandidat utama untuk evaluasi kontrak.  
- **Amazon Logistics** menunjukkan beberapa “anomali positif” → waktu transit bervariasi (1–9 hari) tapi status tetap “On Time”.  
- Jalur **NYC → Boston** paling efisien (cost $45, 4 hari transit).  
- Jalur **SEA → Boston** paling mahal ($280,88) walau waktu normal.  
- Total biaya keseluruhan **$317.722** untuk **6.865 hari transit**, menjadi baseline efisiensi biaya.

---

## 💡 Kesimpulan Utama
| Fokus | Temuan | Rekomendasi |
|--------|---------|-------------|
| **Efisiensi Carrier** | DHL mahal tapi cepat, USPS paling hemat | Optimalkan kombinasi carrier berdasarkan rute |
| **Warehouse** | SF & LA paling produktif, namun biaya tinggi | Evaluasi distribusi volume antar warehouse |
| **Musim & Permintaan** | Peak di Mei & November | Rencanakan kapasitas & biaya di high season |
| **Anomali Biaya** | Rute SEA–Boston terlalu mahal | Audit kontrak & optimalkan rute alternatif |
| **Kinerja Umum** | 94,5% pengiriman tepat waktu | Pertahankan SLA & pemantauan via dashboard |

---

## ⚙️ Teknologi & Tools
| Tahap | Teknologi |
|--------|------------|
| Data Processing | Python (`pandas`, `numpy`, `matplotlib`, `seaborn`) |
| Data Cleaning | IQR Outlier Removal, Consistency Check |
| Visualization | Power BI |
| Output | 3 halaman dashboard interaktif: Performance, Efficiency, Anomaly |
| Dataset | 1.640 shipment record, 8 kolom utama |

---

## 📈 Cuplikan Dashboard
<p align="center">
  <img src="https://github.com/yazidabd/Logistics-DA-project/blob/main/assest/Logistics_Dasb_P1.png" width="750"><br>
  <img src="https://github.com/yazidabd/Logistics-DA-project/blob/main/assest/Logistics_Dasb_P2.png" width="750"><br>
  <img src="https://github.com/yazidabd/Logistics-DA-project/blob/main/assest/Logistics_Dasb_P3.png" width="750">
</p>

---

## 💼 Dampak Bisnis
✅ Mempercepat proses **evaluasi kinerja carrier & warehouse**  
✅ Mengidentifikasi peluang **optimasi biaya rute**  
✅ Menjadi alat **monitoring KPI logistik real-time**  
✅ Mendukung keputusan berbasis data untuk efisiensi operasional

---

## 👤 Author
**Yazid Abdullah Subhi**  
_Data Enthusiast | data analyst & data science_  
📧 [yazidabd@gmail.com](mailto:yazidabd@gmail.com)
