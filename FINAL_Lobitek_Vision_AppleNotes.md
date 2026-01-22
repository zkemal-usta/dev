# LOBITEK VISION - TEKNİK PLANLAMA VE MATEMATİK
*(Bu doküman Apple Notes ile tam uyumludur. Önizlemeden kopyalayıp yapıştırabilirsiniz.)*

---

## 1. STEREO VİZYON VE DERİNLİK MATEMATİĞİ

### 1.1. Pinhole (İğne Deliği) Kamera Modeli
Kameranın matematiksel modeli şu matris çarpımı ile ifade edilir:
**(3B Nokta)** P(X,Y,Z)  ⟶  **(2B Piksel)** p(u,v)

**Temel Denklem:**
> s · p  =  K · [R | t] · P

Burada:
*   **s**: Ölçek faktörü (Scale)
*   **K**: İç Parametre Matrisi (Intrinsic)
*   **[R | t]**: Dış Parametreler (Rotasyon ve Öteleme)

**K Matrisi (İç Parametreler):**
>     ⎡ fₓ   0    cₓ ⎤
> K = ⎢ 0    fᵧ   cᵧ ⎥
>     ⎣ 0    0    1  ⎦

*   **fₓ, fᵧ**: Odak uzaklıkları (Focal Length)
*   **cₓ, cᵧ**: Görüntü merkezi (Principal Point)

---

### 1.2. Derinlik (Z) Hesabı
Derinlik algısı, iki kamera arasındaki görüntü kaymasına (**Disparity**) dayanır.

*   **B**: Baseline (Kameralar arası mesafe)
*   **d**: Disparity (Sol ve Sağ görüntüdeki piksel farkı)
*   **f**: Odak uzaklığı

**Disparity Formülü:**
> d = x_sol - x_sağ

**Derinlik (Z) Formülü:**
> Z = (f · B) / d

**Hata Analizi:**
Derinlik hatası (ΔZ), mesafenin karesiyle artar:
> |ΔZ| = (Z² / (f · B)) · Δd

*   **Sonuç:** Nesne uzaklaştıkça (Z arttıkça), hata payı karesel (Z²) olarak büyür.

---

### 1.3. 3 Kameralı Sistem (Trifocal)
3. kamera dikey eksende ekstra bir doğrulama sağlar.
*   **Zₕ**: Yatay kameralardan gelen derinlik
*   **Zᵥ**: Dikey kameradan gelen derinlik

Final derinlik, güvenilirlik ağırlıkları (w) ile birleştirilir:
> Z_final = (wₕ·Zₕ + wᵥ·Zᵥ) / (wₕ + wᵥ)

---

## 2. ENGEL ALGILAMA VE HARİTALAMA

### 2.1. Nokta Bulutu (3B Dönüşüm)
Eldeki 2B pikselden ve Z bilgisinden gerçek dünya koordinatlarına dönüş:

> X = (u - cₓ) · Z / fₓ
> Y = (v - cᵧ) · Z / fᵧ
> Z = Z

### 2.2. Zemin Filtreleme (Ground Plane)
Yükseklik (Y) değerine göre engel tespiti yapılır:
*   Eğer **Y > h_eşik**  ⟶  **ENGEL**
*   Eğer **Y ≤ h_eşik**  ⟶  **ZEMİN (YOL)**

### 2.3. Maliyet Haritası (Costmap)
Robotun engele çarpmaması için engel etrafına sanal bir güvenlik alanı çizilir.
> Cost(r) = e^(-α · (r - R_robot))

*   Engele yaklaştıkça maliyet (tehlike) **üstel olarak (exponential)** artar.

---

## 3. ENGELDEN KAÇMA (APF & DWA)

### 3.1. Yapay Potansiyel Alanlar (APF)
Robot, fiziksel kuvvetlerin etkisindeymiş gibi hareket eder.
**Toplam Kuvvet = Çekici Kuvvet + İtici Kuvvet**

1.  **Çekici (Attractive):** Robotu hedefe çeker.
    > F_att = -k_att · (Pozisyon - Hedef)

2.  **İtici (Repulsive):** Robotu engelden iter.
    > F_rep = k_rep · (1/d - 1/d₀) · (1/d²)

### 3.2. Dinamik Pencere Yaklaşımı (DWA)
Robotun hızı (v) ve dönüşü (ω) için en iyi ikili seçilir.
Puanlama Fonksiyonu:
> G(v,ω) = α · (Hedef_Yönü) + β · (Engel_Mesafesi) + γ · (Hız)

---

## 4. OTO ROTA PLANLAMA (A*)

### 4.1. A* (A-Star) Algoritması
En kısa yolu bulan algoritmanın temel formülü:
> f(n) = g(n) + h(n)

*   **g(n):** Başlangıçtan buraya kadar gelmenin maliyeti.
*   **h(n):** Buradan hedefe kuş uçuşu tahmini maliyet (**Heuristic**).

**Öklid (Euclidean) Heuristic:**
> h(n) = √((x - x_hedef)² + (y - y_hedef)²)

### 4.2. Rota Yumuşatma (Bezier)
Keskin dönüşleri yumuşatmak için 2. derece Bezier eğrisi:
> B(t) = (1-t)²·P₀ + 2(1-t)t·P₁ + t²·P₂

*   **t**: 0 ile 1 arasında değişen zaman parametresi.
