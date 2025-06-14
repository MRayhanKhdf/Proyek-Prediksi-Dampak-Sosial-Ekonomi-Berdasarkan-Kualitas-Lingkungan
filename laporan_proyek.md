
# Laporan Proyek Machine Learning - [Muhammad Rayhan Khadafi]

## Domain Proyek

Polusi udara dan polusi air di perkotaan merupakan faktor penting yang memengaruhi kualitas hidup masyarakat dan perkembangan ekonomi. Partikulat halus (PM2.5) yang diukur oleh WHO, bersama dengan tingkat polusi air, bisa menjadi indikasi daya dukung lingkungan suatu kota. Penelitian ini menggabungkan indikator lingkungan (UEQ Index) dengan variabel ekonomi (GDP per kapita) untuk memprediksi status sosial-ekonomi kota.

**Referensi Utama**:
1. D. Paul, K. Mukherjee, J. Pandey, and A. Dutta Roy, “Assessing the Urban Environmental Quality: A Case Study of Kolkata Metropolitan Area, India,” *IOP Conference Series*, vol. 1164, no. 1, p. 012001, Apr. 2023. doi: 10.1088/1755-1315/1164/1/012001.
2. L. Tan et al., “Assessment of Urban Environmental Quality by Socioeconomic and Environmental Variables Using Open‐Source Datasets,” *Transactions in GIS*, Sep. 2024. doi: 10.1111/tgis.13250.

## Business Understanding

### Problem Statements
1. Bagaimana kombinasi kualitas udara (PM2.5) dan kualitas air memengaruhi GDP per kapita suatu kota?
2. Dapatkah kita secara akurat mengklasifikasikan kota berdasarkan status sosial-ekonomi (tinggi/rendah GDP) menggunakan indikator lingkungan?

### Goals
1. Mengukur hubungan antara UEQ_Index, WaterPollution, dan GDP per kapita.
2. Membangun model klasifikasi untuk memprediksi kelas sosial-ekonomi (SocioClass).
3. Membangun model regresi untuk memprediksi nilai kontinu GDP per kapita.

### Solution Statements
- Membuat **UEQ Index** dengan menggabungkan normalisasi AirQuality dan PM2.5.
- Menggunakan algoritma klasifikasi: Logistic Regression, Random Forest Classifier, dan KNN.
- Melakukan hyperparameter tuning pada Random Forest Classifier.
- Membandingkan performa model klasifikasi berdasarkan accuracy, precision, recall, dan F1-score.
- Menggunakan Linear Regression dan Random Forest Regressor untuk prediksi kontinu.
- Evaluasi regresi menggunakan MAE dan R².

## Data Understanding

Dataset yang digunakan terdiri dari 3 sumber:
- https://www.kaggle.com/datasets/patricklford/water-and-air-quality?select=Cities1.csv
- https://www.kaggle.com/datasets/patricklford/water-and-air-quality?select=WHO_PM.csv
- https://www.kaggle.com/datasets/zgrcemta/world-gdpgdp-gdp-per-capita-and-annual-growths?select=gdp_per_capita.csv

Data yang digunakan dari sumber adalah:
1. **Cities1.csv**: Kolom: `City`, `Region`, `Country`, `AirQuality`, `WaterPollution`.
Total baris 3963
Total Kolom 5

2. **WHO_PM.csv**: Kolom: `IndicatorCode`, `Indicator`, `ValueType`, `ParentLocationCode`, `ParentLocation`, `LocationType`, `SpatialDimValueCode`, `Location`, `PeriodType`, `Period`, `Value` (PM2.5).
Total baris 9450
Total Kolom 34

3. **gdp_per_capita.csv**: Kolom: `Country Name`, `Code`, dan nilai GDP per kapita untuk tahun 1960–2020.
Total baris 266
Total Kolom 64

Jumlah data tergantung cities dan periode (2014-2019).

Deskripsi variabel:
- `City`: Nama kota.
- `Region`: Wilayah geografis kota (mis. Asia Timur, Eropa, dll).
- `Country`: Negara kota.
- `AirQuality`: Indeks kualitas udara lokal.
- `WaterPollution`: Indeks polusi air lokal.
- `Value`: Konsentrasi PM2.5 (µg/m³) dari WHO.
- `UEQ_Index`: Rata-rata nilai normalisasi `AirQuality` dan `PM25`.
- `GDP_2019`: Nilai GDP per kapita tahun 2019 (USD).
- `SocioClass`: Label klasifikasi: 1 jika `GDP_2019` > median, else 0.

**Exploratory Data Analysis**:
- Statistik deskriptif menunjukkan sebaran `AirQuality`, `WaterPollution`, dan `Value`.
- Histogram dan scatter plot membantu melihat distribusi dan korelasi awal antara fitur lingkungan.

**Kondisi Data**:
- Pada Cities1.csv terdapat 425 missing data di kolom `Region`
- Pada WHO_PM.csv Terdapat data kosong, namun data kosong ada pada kolom yang tidak digunakan, hingga pada kolom data yang digunakan tidak ada yang kehilangan
- Pada gdp_per_capita.csv, terdapat data missing pada bagian tahun. Dalam rentang kolom tahun dari 1960 hingga 2020, terdapat 266 total data missing.
- Pada gdp_per_capita.csv terdapat kolom code, itu berguna sebagai identifier negara. Jadi code itu adalah code negara
- Pada gdp_per_capita.cs terdapat Unnamed: 65 yang berisi Kolom kosong tanpa nama atau informasi yang valid; didapat dri kaggle dan tidak ada penjelasan mengenai itu hingga diabaikan dalam analisis.
- Pada WHO_PM.csv terdapat beberapa kolom lagi, yaitu:
  - IndicatorCode: Kode indikator (dalam hal ini semua SDGPM25 – konsentrasi PM2.5).
  - Indicator: Deskripsi indikator (PM2.5 – partikel halus yang berbahaya jika terhirup).
  - ValueType: Jenis nilai (semuanya bertipe teks).
  - ParentLocationCode / ParentLocation: Kode dan nama kawasan/region yang lebih besar (misalnya, "AFR" untuk Afrika).
  - Location / SpatialDimValueCode: Nama negara dan kode tiga huruf (misalnya, "KEN" untuk Kenya).
  - Period: Tahun pengukuran.
  - Dim1 type, Dim1, Dim1ValueCode: Dimensi tambahan 1 dan kodenya (tidak relevan untuk PM2.5).
  - Dim2 type, Dim2, Dim2ValueCode: Dimensi tambahan 2.
  - Dim3 type, Dim3, Dim3ValueCode: Dimensi tambahan 3.
  - DataSourceDimValueCode, DataSource: Kode dan nama sumber data.
  - FactValueNumericLow / High: Rentang nilai estimasi konsentrasi PM2.5 (bawah dan atas).
  - Value: Nilai utama (rata-rata) dan rentangnya (contoh: "10.01 [6.29–13.74]").
  - FactValueUoM: Satuan dari nilai (umumnya µg/m3).
  - FactValueNumericPrefix, FactValueNumericLowPrefix, FactValueNumericHighPrefix: Prefix nilai minimum/maksimum.
  - FactComments: Komentar terkait fakta/nilai.
  - Language: Bahasa data (biasanya en).
  - DateModified: Tanggal terakhir data dimodifikasi.
  - IsLatestYear: Menunjukkan apakah data tersebut merupakan data tahun terbaru untuk lokasi tersebut.

## Data Preparation

1. **Penggabungan Data**  
   - Menggabungkan `Cities1.csv` dan `WHO_PM.csv` berdasarkan kolom `City` = `Location`.
2. **Normalisasi Data dan Pembersihan Data Missing**  
   - Menggunakan fillna() Semua nilai kosong (NaN) pada kolom PM25 akan diganti dengan nilai rata-rata dari kolom tersebut menggunakan .mean()
   - `AirQuality_norm` = (AirQuality – min) / (max – min).  
   - `PM25` diambil dari kolom `Value`, missing values diisi dengan rata-rata, kemudian dinormalisasi → `PM25_norm`.
   - `UEQ_Index` = (AirQuality_norm + PM25_norm) / 2.
3. **Integrasi GDP per Kapita**  
   - Ekstrak kolom `Country Name` dan `2019` pada `gdp_per_capita.csv` → `Country`, `GDP_2019`.
   - Gabungkan dengan data utama berdasarkan `Country`.
4. **Label Klasifikasi**  
   - Buat `SocioClass` berdasarkan median `GDP_2019` (1 if > median, else 0).
5. **Split Data**  
   - Train-test split (80:20) untuk fitur `UEQ_Index`, `WaterPollution`, `Region`, `Country` dan target `SocioClass` (klasifikasi) serta `GDP_2019` (regresi).
6. **One-Hot Encoding & StandardScaler**  
   - Kategorikal: `Region`, `Country` menggunakan OneHotEncoder.  
   - Numerik: `UEQ_Index`, `WaterPollution` menggunakan StandardScaler.

## Modeling

### Model Klasifikasi
- **Logistic Regression**  
  - Model sederhana, cepat, dan mudah diinterpretasi. Cocok sebagai baseline.
  - Cara kerja: Logistic Regression memodelkan probabilitas suatu kelas sebagai fungsi logistik dari kombinasi linier fitur. Rumus dasarnya adalah:
   〖P(y+1|x)〗^ =1/(1+e^(-(β_0+β_1 x_1+⋯+β_n β_n)) )
  - Kemudian diklasifikasikan ke kelas 1 jika probabilitas di atas ambang batas (default = 0.5), jika tidak maka kelas 0.
  - Memiliki parameter max_iter=1000 yang berfungsi untuk memperbesar Iterasi maksimum agar model konvergen.
- **Random Forest Classifier**  
  - Robust terhadap outlier dan mampu menangkap hubungan non-linear.
  - Cara kerja: Random Forest adalah ensemble dari banyak decision tree. Setiap tree dilatih dari sampel acak (dengan pengambilan ulang) dan subset fitur acak. Prediksi klasifikasi dilakukan dengan voting mayoritas dari semua pohon.
- **KNN Classifier**  
  - Model berbasis jarak, berkinerja baik jika data terdistribusi merata.
  - Mengklasifikasikan titik baru berdasarkan mayoritas label dari k tetangga terdekat (berdasarkan jarak, biasanya Euclidean).
  - Menggunakan model KNeighborsClassifier() dengan parameter default n_neighbors=5 yaitu mencari 5 data terdekat
- **Hyperparameter Tuning**  
  - Hyperparameter tuning dilakukan pada model Random Forest Classifier menggunakan GridSearchCV. Tiga parameter yang diuji adalah n_estimators (50, 100, 200), max_depth (None, 10, 20), dan min_samples_split (2, 5). Evaluasi dilakukan menggunakan 5-fold cross-validation dengan metrik akurasi. Hasil tuning menunjukkan bahwa kombinasi parameter terbaik adalah n_estimators=200, max_depth=20, dan min_samples_split=2, yang menghasilkan akurasi tertinggi pada data uji.
  - GridSearchCV diterapkan pada Random Forest:  
  - classifier__n_estimators': [50, 100, 200], Banyaknya pohon (trees)
  - classifier__max_depth': [None, 10, 20], Depth maksimum tiap pohon
  - classifier__min_samples_split': [2, 5], Minimum jumlah sampel agar node dibagi (split)
  - ('preprocessor', preprocessor), Tahap preprocessing (misalnya scaling, encoding)
  - ('classifier', RandomForestClassifier(...)),  Model klasifikasi Random Forest
  - estimator=pipe_rf_tune, Pipeline yang ingin dituning
  - param_grid=rf_param_grid, Grid hyperparameter
  - cv=5, 5-fold cross-validation
  - n_jobs=-1, Menggunakan semua core CPU untuk mempercepat
  - scoring='accuracy', Skor yang digunakan untuk memilih hasil terbaik
  
### Model Regresi
- **Linear Regression**  
  - Mudah diinterpretasi, cocok jika hubungan linier dominan.
  - Memprediksi nilai kontinu berdasarkan kombinasi linier dari fitur: y=β_0+β_1 x_1+⋯+β_n β_n+ε
  - Model mencoba meminimalkan Mean Squared Error antara prediksi dan target aktual.
  - Menggunkan model LinearRegression() dan menggunakan semua parameter default tanpa perubahan(fit_intercept=True, normalize='deprecated', dll).
- **Random Forest Regressor**  
  - Mengatasi non-linearitas, lebih akurat tapi kurang interpretabel.
  - Seperti Random Forest Classifier, tapi outputnya adalah rata-rata hasil prediksi dari semua decision tree (regresi).
  - Mengunakan model RandomForestRegressor(n_estimators=100, random_state=42) dengan parameternya:
    - n_estimators=100: Jumlah pohon dalam ensemble.
	- random_state=42: digunakan agar hasil acak dan memastikan hasil replikasi yang konsisten.
	
## Evaluation

### Klasifikasi Metrics
- **Accuracy**: Proporsi prediksi benar.  
- **Precision, Recall, F1-Score**:  
  - Precision (TP / (TP+FP)), recall (TP / (TP+FN)), F1 (harmonik rata-rata).  
- **Confusion Matrix**: Menampilkan TP, TN, FP, FN.

### Regresi Metrics
- **Mean Absolute Error (MAE)**:  
  $$\[ MAE = \frac{1}{n} \sum_{i=1}^n |y_i - \hat{y}_i| \]$$
- **R-squared (R²)**:  
  $$\[ R^2 = 1 - \frac{\sum_{i=1}^n (y_i - \hat{y}_i)^2}{\sum_{i=1}^n (y_i - \bar{y})^2} \]$$

#### Hasil Klasifikasi
- **Logistic Regression**: Akurasi ≈ 99 %, F1-score untuk masing-masing kelas ≈ 0.99.
- **Random Forest**: Akurasi ≈ 99 %, hasil setelah tuning menunjukkan precision/recall ≈ 0.99.
- **KNN**: Akurasi ≈ 98 %, sedikit lebih rendah dibanding dua model lainnya.
- **Model Terbaik**: Random Forest Classifier (GridSearchCV) dengan akurasi ≈ 99.19 %.

#### Hasil Regresi
- **Linear Regression**: MAE ≈ 19,575.65, R² ≈ 0.075.  
- **Random Forest Regressor**: MAE ≈ 12,304.70, R² ≈ 0.612.  
- **Model Terbaik**: Random Forest Regressor mampu menjelaskan 61 % variansi GDP per kapita.

## Kesimpulan dan Rekomendasi

1. **Ringkasan Hasil**  
   - Hubungan kuat antara kualitas lingkungan (UEQ_Index) dan status sosial-ekonomi.  
   - Random Forest Classifier dan Regressor adalah model terbaik untuk klasifikasi dan regresi.

2. **Interpretasi Model**  
   - Kota dengan UEQ_Index yang rendah dan tingkat polusi air tinggi cenderung memiliki GDP per kapita di bawah median.  
   - Random Forest menangkap pola non-linear yang tidak dapat dijelaskan Linear Regression.

3. **Saran Kebijakan**  
   - Tingkatkan upaya pengendalian polusi udara (regulasi emisi) dan pemantauan kualitas air.  
   - Investasi infrastruktur ramah lingkungan untuk meningkatkan UEQ_Index, yang pada gilirannya dapat mendorong pertumbuhan ekonomi.  
   - Edukasi masyarakat terkait dampak polusi terhadap produktivitas dan kesehatan.

4. **Penelitian Selanjutnya**  
   - Tambahkan variabel lain (pendidikan, urbanisasi) untuk meningkatkan akurasi.  
   - Analisis panel data untuk tren jangka panjang kualitas lingkungan dan GDP.  


---

**Referensi**  
[1] D. Paul, K. Mukherjee, J. Pandey, and A. Dutta Roy, “Assessing the Urban Environmental Quality: A Case Study of Kolkata Metropolitan Area, India,” *IOP Conference Series*, vol. 1164, no. 1, p. 012001, Apr. 2023. doi: 10.1088/1755-1315/1164/1/012001.  
[2] L. Tan et al., “Assessment of Urban Environmental Quality by Socioeconomic and Environmental Variables Using Open‐Source Datasets,” *Transactions in GIS*, Sep. 2024. doi: 10.1111/tgis.13250.
