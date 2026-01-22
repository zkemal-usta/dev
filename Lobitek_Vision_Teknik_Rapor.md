# Lobitek Vision - Teknik Rapor
## 3 Kameralı Stereo Derinlik, Engel Algılama ve Rota Planlama

> **Not:** Bu belgeyi en iyi şekilde görüntülemek için VS Code içinde **`Cmd + Shift + V`** (Mac) veya **`Ctrl + Shift + V`** (Windows) kısayolu ile Önizleme (Preview) modunu açın. Matematiksel formüller otomatik olarak render edilecektir. Ardından sağ tıklayıp "Yazdır" diyerek PDF olarak kaydedip Apple Notes'a atabilirsiniz.

---

## 1. 3 Kameralı Stereo Vizyon ve Derinlik Matematiği

### 1.1. Giriş
Bu bölüm, 3 kameralı sistem (Trifocal Tensor veya Multi-Baseline Stereo) kullanarak derinlik kestirimi, kamera kalibrasyonu ve üçgenleme (triangulation) prensiplerini detaylandırır.

### 1.2. Pinhole Kamera Modeli (İğne Deliği Modeli)
Her bir kamera için temel izdüşüm matrisi aşağıdaki gibi tanımlanır. 3B uzaydaki bir nokta $P(X, Y, Z)$ ve görüntü düzlemindeki karşılığı $p(u, v)$ olsun.

$$
s \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = K [R | t] \begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}
$$

Burada:
- $s$: Ölçek faktörü (derinlik ile orantılı).
- $K$: İç (Intrinsic) kamera parametre matrisi.
- $R$: Rotasyon matrisi ($3 \times 3$).
- $t$: Öteleme (Translation) vektörü.

**İç Parametre Matrisi ($K$):**
$$
K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}
$$
- $f_x, f_y$: Odak uzaklıkları (piksel cinsinden).
- $c_x, c_y$: Görüntü merkezi (ana nokta).

### 1.3. Epipolar Geometri ve Disparity (Uyumsuzluk)
İki kamera (Stereo Çifti) arasındaki temel ilişki *disparity* kavramına dayanır.
Sol kamera ile sağ kamera arasındaki yatay mesafe (Baseline) $B$ olsun.
Bir $P$ noktasının sol görüntüdeki x koordinatı $x_L$ ve sağ görüntüdeki x koordinatı $x_R$ ise, disparity $d$:

$$
d = x_L - x_R
$$

**Derinlik ($Z$) Hesabı:**
Benzer üçgenler teoremi kullanılarak derinlik $Z$ şu şekilde hesaplanır:

$$
\frac{Z}{f} = \frac{B}{d} \implies Z = \frac{f \cdot B}{d}
$$

Burada:
- $Z$: Kameraya olan dik mesafe (Derinlik).
- $f$: Odak uzaklığı.
- $B$: İki kamera merkezi arasındaki mesafe (Baseline).
- $d$: Disparity değeri.

> **Önemli Not:** Disparity $d$, derinlik $Z$ ile ters orantılıdır. $d \to 0$ iken $Z \to \infty$. Bu nedenle uzak nesnelerde derinlik hassasiyeti düşer.

### 1.4. 3 Kameralı (Trifocal) Sistem Avantajı
İki kameralı sistemlerde yatay kenarların eşleştirilmesi zordur (Aperture problem). 3. bir kamera (genellikle üstte) dikey disparity bilgisi de sağlar.

Kameralar $C_1$ (Sol), $C_2$ (Sağ), $C_3$ (Üst) olsun.
Elde edilen iki derinlik tahmini ($Z_h$ ve $Z_v$) birleştirilerek hata minimize edilir (Kalman Filtresi veya Ağırlıklı Ortalama):

$$
Z_{final} = \frac{w_h \cdot Z_h + w_v \cdot Z_v}{w_h + w_v}
$$

### 1.5. Hata Analizi
Derinlik ölçümündeki hata $\Delta Z$, disparity hatasına $\Delta d$ bağlıdır:

$$
\left| \Delta Z \right| = \left| -\frac{f \cdot B}{d^2} \Delta d \right| = \frac{Z^2}{f \cdot B} \Delta d
$$

**Yorum:** Hata, derinliğin karesi ($Z^2$) ile artar.

---

## 2. Engel Algılama ve Haritalama

### 2.1. Nokta Bulutu (Point Cloud) Dönüşümü
Stereo kameradan elde edilen derinlik haritası, robotun lokal koordinat sisteminde 3B nokta bulutuna dönüştürülür.
Bir piksel $(u, v)$ ve derinliği $Z$ için 3B koordinatlar:

$$
X = \frac{(u - c_x) \cdot Z}{f_x}, \quad Y = \frac{(v - c_y) \cdot Z}{f_y}, \quad Z = Z
$$

### 2.2. Zemin Düzlemi Filtreleme
Engelleri tespit edebilmek için zeminin (yolun) ayırt edilmesi gerekir. Basit yükseklik filtresi:

$$
Engel(P) = \begin{cases} 
1 (EVET), & \text{eğer } P_y > h_{eşik} \\
0 (HAYIR), & \text{eğer } P_y \leq h_{eşik}
\end{cases}
$$

### 2.3. Doluluk Haritası (Occupancy Grid)
3B uzay, 2B kuş bakışı bir ızgaraya indirgenir.
**Log-Odds Güncelleme Kuralı:**
$l_{t,i}$ hücrenin $t$ anındaki log-odds değeri olsun.

$$
l_{t,i} = l_{t-1,i} + \text{inverse\_sensor\_model}(z_t) - l_0
$$

Burada $l(x) = \log \left( \frac{P(x)}{1 - P(x)} \right)$ dir.

### 2.4. Güvenlik Çemberi ve Maliyet Haritası (Costmap)
Robotun fiziksel boyutlarını hesaba katmak için engeller haritada "şişirilir".
Maliyet Fonksiyonu $C(dist)$:

$$
C(dist) = \begin{cases} 
\text{LETHAL}, & dist \leq R_{robot} \\
\exp(-\alpha \cdot (dist - R_{robot})), & R_{robot} < dist \leq R_{inflation} \\
0, & dist > R_{inflation}
\end{cases}
$$

---

## 3. Engelden Kaçma Algoritmaları

### 3.1. Yapay Potansiyel Alanlar (APF)
Robot, hedefe doğru çeken ve engellerden iten sanal kuvvetlerin etkisi altındaymış gibi hareket eder.

**Toplam Potansiyel:**
$$
U_{total}(q) = U_{att}(q) + U_{rep}(q)
$$

**Çekici Potansiyel (Attractive):**
Hedefe ($q_{goal}$) ulaşmak için parabolik kuyu modeli:
$$
U_{att}(q) = \frac{1}{2} k_{att} \cdot d(q, q_{goal})^2
$$
Kuvvet $F_{att} = -\nabla U_{att}(q) = -k_{att} \cdot (q - q_{goal})$

**İtici Potansiyel (Repulsive):**
$$
U_{rep}(q) = \begin{cases} 
\frac{1}{2} k_{rep} \left( \frac{1}{d(q, q_{obs})} - \frac{1}{d_0} \right)^2, & \text{eğer } d \leq d_0 \\
0, & \text{eğer } d > d_0 
\end{cases}
$$

### 3.2. Dinamik Pencere Yaklaşımı (DWA)
Robotun kinematik sınırlarını ve hızını hesaba katarak en iyi hız çiftini $(v, \omega)$ seçer.

**Amaç Fonksiyonu $G(v, \omega)$:**
$$
G(v, \omega) = \alpha \cdot \text{heading}(v, \omega) + \beta \cdot \text{dist}(v, \omega) + \gamma \cdot \text{vel}(v, \omega)
$$
- **heading:** Hedefe yönelim.
- **dist:** Engele mesafe.
- **vel:** Robot hızı.

---

## 4. Otomatik Rota Planlama

### 4.1. A* (A-Star) Algoritması
Izgara tabanlı haritalarda en kısa yolu bulmak için kullanılır.
Düğüm maliyeti:
$$
f(n) = g(n) + h(n)
$$
- $g(n)$: Başlangıçtan maliyet.
- $h(n)$: Hedefe tahmini maliyet (Heuristic).

**Heuristic (Öklid):**
$$
h(n) = \sqrt{(x_n - x_{hedef})^2 + (y_n - y_{hedef})^2}
$$

### 4.2. RRT (Rapidly-exploring Random Tree)
Yüksek boyutlu uzaylarda rastgele örnekleme ile yol bulur.
Ağaç $T$, rastgele seçilen $q_{rand}$ noktasına doğru sürekli genişletilir.

### 4.3. Rota Düzgünleştirme (Path Smoothing)
A* veya RRT çıktısını yumuşatmak için Bezier Eğrileri kullanılır:

$$
B(t) = (1-t)^2 P_0 + 2(1-t)t P_1 + t^2 P_2, \quad t \in [0, 1]
$$
