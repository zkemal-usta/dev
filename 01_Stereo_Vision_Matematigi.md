# 3 Kameralı Stereo Vizyon ve Derinlik Matematiği

## 1. Giriş
Bu belge, 3 kameralı sistem (Trifocal Tensor veya Multi-Baseline Stereo) kullanarak derinlik kestirimi, kamera kalibrasyonu ve üçgenleme (triangulation) prensiplerini detaylandırır.

## 2. Pinhole Kamera Modeli (İğne Deliği Modeli)
Her bir kamera için temel izdüşüm matrisi aşağıdaki gibi tanımlanır. 3B uzaydaki bir nokta $P(X, Y, Z)$ ve görüntü düzlemindeki karşılığı $p(u, v)$ olsun.

$$
s \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} = K [R | t] \begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}
$$

Burada:
- $s$: Ölçek faktörü (derinlik ile orantılı).
- $K$: İç (Intrinsic) kamera parametre matrisi.
- $R$: Rotasyon matrisi ($3 \times 3$).
- $t$: Öteleme (Translation) vektörü.

### İç Parametre Matrisi ($K$)
$$
K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}
$$
- $f_x, f_y$: Odak uzaklıkları (piksel cinsinden).
- $c_x, c_y$: Görüntü merkezi (ana nokta).

## 3. Epipolar Geometri ve Disparity (Uyumsuzluk)
İki kamera (Stereo Çifti) arasındaki temel ilişki *disparity* kavramına dayanır.

Sol kamera ile sağ kamera arasındaki yatay mesafe (Baseline) $B$ olsun.
Bir $P$ noktasının sol görüntüdeki x koordinatı $x_L$ ve sağ görüntüdeki x koordinatı $x_R$ ise, disparity $d$:

$$
d = x_L - x_R
$$

### Derinlik ($Z$) Hesabı

Benzer üçgenler teoremi kullanılarak derinlik $Z$ şu şekilde hesaplanır:

$$
\frac{Z}{f} = \frac{B}{d} \implies Z = \frac{f \cdot B}{d}
$$

Burada:
- $Z$: Kameraya olan dik mesafe (Derinlik).
- $f$: Odak uzaklığı.
- $B$: İki kamera merkezi arasındaki mesafe (Baseline).
- $d$: Disparity değeri ($x_L - x_R$).

**Önemli Not:** Disparity $d$, derinlik $Z$ ile ters orantılıdır. $d \to 0$ iken $Z \to \infty$.

## 4. 3 Kameralı (Trifocal/Multi-View) Sistem Avantajı
İki kameralı sistemlerde yatay kenarların eşleştirilmesi zordur (Aperture problem). 3. bir kamera (genellikle üstte veya L şeklinde dizilimde) dikey disparity bilgisi de sağlar veya doğrulama (validation) imkanı sunar.

### 3 Kamera Konfigürasyonu
Kameralar $C_1$ (Sol), $C_2$ (Sağ), $C_3$ (Üst) olsun.
- **Yatay Baseline ($B_h$):** $C_1 - C_2$ arası.
- **Dikey Baseline ($B_v$):** $C_1 - C_3$ arası.

Elde edilen iki derinlik tahmini ($Z_h$ ve $Z_v$) birleştirilerek hata minimize edilir (Kalman Filtresi veya Ağırlıklı Ortalama):

$$
Z_{final} = \frac{w_h \cdot Z_h + w_v \cdot Z_v}{w_h + w_v}
$$

## 5. Hata Analizi ve Belirsizlik
Derinlik ölçümündeki hata $\Delta Z$, disparity hatasına $\Delta d$ bağlıdır:

$$
Z = \frac{f \cdot B}{d}
$$

Türev alırsak:

$$
\left| \Delta Z \right| = \left| \frac{\partial Z}{\partial d} \Delta d \right| = \left| -\frac{f \cdot B}{d^2} \Delta d \right| = \frac{Z^2}{f \cdot B} \Delta d
$$

**Yorum:** Hata, derinliğin karesi ($Z^2$) ile artar. Bu nedenle uzak nesnelerde hata payı çok daha yüksektir. Hatayı azaltmak için:
1. $B$ (Baseline) artırılmalıdır.
2. $f$ (Odak uzaklığı) artırılmalıdır (daha dar görüş açısı).
3. Sub-pixel interpolasyon ile $\Delta d$ düşürülmelidir.
