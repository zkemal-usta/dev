# Engel Algılama ve Haritalama (Obstacle Detection)

## 1. Nokta Bulutu (Point Cloud) Dönüşümü
Stereo kameradan elde edilen derinlik haritası, robotun lokal koordinat sisteminde 3B nokta bulutuna dönüştürülür.
Bir piksel $(u, v)$ ve derinliği $Z$ için 3B koordinatlar:

$$
X = \frac{(u - c_x) \cdot Z}{f_x}
$$
$$
Y = \frac{(v - c_y) \cdot Z}{f_y}
$$
$$
Z = Z
$$

## 2. Zemin Düzlemi Filtreleme (Ground Plane Segmentation)
Engelleri tespit edebilmek için zeminin (yolun) ayırt edilmesi gerekir. Düz bir zeminde robotun yüksekliği $h_{cam}$ ve eğim açısı $\theta$ bellidir.
Bir noktanın $(X, Y, Z)$ zemin olup olmadığına karar vermek için RANSAC düzlem uydurma veya yükseklik eşik değeri kullanılır.

Basit yükseklik filtresi:
Robot koordinat sisteminde $Y$ ekseni yukarıyı gösteriyorsa (veya $Z$ ekseni yüksekliği temsil ediyorsa):

$$
Engel\_Misin(P) = \begin{cases} 
1 (EVET), & \text{eğer } P_y > h_{eşik} \\
0 (HAYIR), & \text{eğer } P_y \leq h_{eşik}
\end{cases}
$$

## 3. Doluluk Haritası (Occupancy Grid Map)
3B uzay, 2B kuş bakışı (Bird's Eye View) bir ızgaraya indirgenir.
Izgara hücresi $m_{i,j}$'nin doluluk olasılığı $P(m_{i,j})$ Bayes Filtresi ile güncellenir.

Olasılık Güncelleme Kuralı (Log-Odds Formülasyonu):
$l_{t,i}$ hücrenin $t$ anındaki log-odds değeri olsun.

$$
l_{t,i} = l_{t-1,i} + \text{inverse\_sensor\_model}(z_t) - l_0
$$

Burada $l(x) = \log \left( \frac{P(x)}{1 - P(x)} \right)$ dir.

## 4. Güvenlik Çemberi ve Şişirme (Inflation Radius)
Robotun fiziksel boyutlarını hesaba katmak için engeller haritada "şişirilir".
Robot yarıçapı $R_{robot}$ ise, her engel noktası etrafında $R_{robot} + \epsilon$ yarıçaplı bir maliyet alanı oluşturulur.

Maliyet Fonksiyonu $C(dist)$:
$$
C(dist) = \begin{cases} 
\text{LETHAL}, & dist \leq R_{robot} \\
\exp(-\alpha \cdot (dist - R_{robot})), & R_{robot} < dist \leq R_{inflation} \\
0, & dist > R_{inflation}
\end{cases}
$$
- $dist$: En yakın engele olan Öklid mesafesi.
- $\alpha$: Maliyet düşüş katsayısı.
