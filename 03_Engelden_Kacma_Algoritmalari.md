# Engelden Kaçma Algoritmaları (Obstacle Avoidance)

## 1. Yapay Potansiyel Alanlar (Artificial Potential Fields - APF)
Robot, hedefe doğru çeken ve engellerden iten sanal kuvvetlerin etkisi altındaymış gibi hareket eder.

### Toplam Potansiyel $U_{total}(q)$
$$
U_{total}(q) = U_{att}(q) + U_{rep}(q)
$$
Burada $q = (x, y)$ robotun konumudur.

### Çekici Potansiyel (Attractive Potential)
Hedefe ($q_{goal}$) ulaşmak için parabolik kuyu modeli kullanılır:
$$
U_{att}(q) = \frac{1}{2} k_{att} \cdot d(q, q_{goal})^2
$$
Kuvvet $F_{att} = -\nabla U_{att}(q) = -k_{att} \cdot (q - q_{goal})$

### İtici Potansiyel (Repulsive Potential)
Engellerden kaçmak için sadece engele yaklaşınca aktif olan bir alan kullanılır:
$$
U_{rep}(q) = \begin{cases} 
\frac{1}{2} k_{rep} \left( \frac{1}{d(q, q_{obs})} - \frac{1}{d_0} \right)^2, & \text{eğer } d(q, q_{obs}) \leq d_0 \\
0, & \text{eğer } d(q, q_{obs}) > d_0 
\end{cases}
$$
- $d_0$: Engel etki mesafesi.
- $k_{rep}$: İtme katsayısı.

### Bileşke Kuvvet
Robotun hareket vektörü:
$$
F_{total} = F_{att} + F_{rep}
$$

## 2. Vektör Alan Histogramı (Vector Field Histogram - VFH)
Potansiyel alanlardaki yerel minimum sorununu çözmek için geliştirilmiştir. 3 aşamalıdır:
1.  **Kartezyen Histogram:** Etrafındaki engelleri ızgara üzerinde işler.
2.  **Kutupsal Histogram:** Robot etrafındaki açısal yoğunluğu ($0^\circ - 360^\circ$) bir histograma dönüştürür.
3.  **Aday Yön Seçimi:** Yoğunluğun eşik değerin altında olduğu "vadiler" (boşluklar) bulunur ve hedefe en yakın olan vadi seçilir.

Açısal sektör $k$ için engel yoğunluğu $h_k$:
$$
h_k = \sum_{i,j \in \text{sektör}_k} m_{i,j}^2 \cdot (a - b \cdot d_{i,j})
$$

## 3. Dinamik Pencere Yaklaşımı (Dynamic Window Approach - DWA)
Robotun kinematik sınırlarını ve hızını hesaba katarak en iyi hız çiftini $(v, \omega)$ seçer.

### Arama Uzayı ($V_r$)
Sadece robotun dinamik sınırları içinde bir sonraki zaman diliminde ulaşabileceği hızlar incelenir.
$$
V_d = \{ (v, \omega) \mid v \in [v_{min}, v_{max}], \omega \in [\omega_{min}, \omega_{max}] \}
$$

### Amaç Fonksiyonu $G(v, \omega)$
Her aday hız çifti için bir skor hesaplanır ve maksimum skoru veren seçilir:
$$
G(v, \omega) = \alpha \cdot \text{heading}(v, \omega) + \beta \cdot \text{dist}(v, \omega) + \gamma \cdot \text{vel}(v, \omega)
$$

- **heading:** Robotun yönünün hedefe ne kadar yakın olduğu.
- **dist:** En yakın engele çarpana kadar gidilebilecek mesafe.
- **vel:** Robotun hızı (hızlı gitmesi tercih edilir).
- $\alpha, \beta, \gamma$: Ağırlık katsayıları.
