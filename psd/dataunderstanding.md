# Data Understanding

## 1. Deskripsi Dataset

Dataset yang digunakan diperoleh dari Kaggle dengan nama McDonald’s Store Reviews dengan jumlah 33.396 records. Dataset ini berisi kumpulan ulasan pelanggan terhadap restoran McDonald’s yang mencakup 10 fitur, di antaranya:

Store Name → Nama toko.
Category → jenis toko.
Store/Location → cabang restoran yang diulas.
Latitude dan Longatitude → lintang bujur letak toko.
Date/Review Time → tanggal ulasan ditulis.
Review Text → ulasan pelanggan dalam bentuk teks bebas.
Rating → skor kepuasan (1–5).

Langkah awal dalam memahami data meliputi:

Mengecek jumlah record dan fitur (dimensi dataset).
Mengidentifikasi missing values atau data duplikat.
Melihat distribusi rating untuk mengetahui kecenderungan review.
Mengeksplorasi teks ulasan untuk menemukan pola kata atau sentimen umum.
Menganalisis data berdasarkan lokasi dan waktu untuk mencari tren.

## Load Data

Data di load dari file csv ke database PostgreSQL menggunakan script Python :

```{code}
import psycopg2
import pandas as pd

df = pd.read_csv("../datasets/McDonald_s_Reviews.csv", encoding="cp1252")

try:
    conn = psycopg2.connect(
        dbname="psd",   # nama database
        user="postgres",          # username PostgreSQL
        password="1123581321",    # password PostgreSQL
        host="localhost",         # host (bisa IP server juga)
        port="5432"               # port default PostgreSQL
    )

    cur = conn.cursor()

    cur.execute("CREATE TABLE reviews (review_id INT PRIMARY KEY, store_name VARCHAR(100), category VARCHAR(100), store_address TEXT, latitude DECIMAL(11, 8), longitude DECIMAL(11, 8), rating_count INT, review_time VARCHAR(100), review TEXT, rating INT);")
    for i in range(len(df)):
        cur.execute("INSERT INTO reviews (review_id, store_name, category, store_address, latitude, longitude, rating_count, review_time, review, rating) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s);", (int(df["reviewer_id"][i]), df["store_name"][i].replace("ý", ""), df["category"][i], df["store_address"][i], float(df["latitude "][i]), float(df["longitude"][i]), int(df["rating_count"][i].replace(",", "")), df["review_time"][0], df["review"][i].replace("ï¿½", "").replace("ï", "").replace("ï¿", "").replace("¿", "").replace("\n", " "), int(df["rating"][i][0])))
    
    conn.commit()

    # tutup koneksi
    cur.close()
    conn.close()
except Exception as e:
    print("Error:", e)
```

Namun sebelum mengeksekusi code python diatas, kita harus pahami terlebih dahulu bentuk data tiap kolom agar pemilihan tipe data pada PostgreSQL tepat dan efisien. Contohnya kolom rating_count yang harusnya bertipe integer (angka) tetapi pada csv berbentuk teks sehingga harus menghapus tanda koma lalu mengkonversi ke integer.

Query membuat tabel :

```{code}
CREATE TABLE reviews (
    review_id INT PRIMARY KEY,
    store_name VARCHAR(100),
    category VARCHAR(100),
    store_address TEXT,
    latitude DECIMAL(11, 8),
    longitude DECIMAL(11, 8),
    rating_count INT,
    review_time VARCHAR(100),
    review TEXT,
    rating INT
);
```

Query untuk insert :

```{code}
INSERT INTO reviews (review_id, store_name, category, store_address, latitude, longitude, rating_count, review_time, review, rating) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s);
```