# Otomatik Rota Planlama (Path Planning)

## 1. Global Rota Planlama vs. Lokal Planlama
- **Global Planlayıcı:** Önceden bilinen harita üzerinde başlangıçtan hedefe en uygun yolu bulur (Statik).
- **Lokal Planlayıcı:** Hareket halindeyken dinamik engellerden kaçınarak global rotayı takip eder (Dinamik).

## 2. A* (A-Star) Algoritması
Izgara tabanlı haritalarda en kısa yolu bulmak için kullanılan en yaygın algoritmadır. Dijkstra algoritmasının sezgisel (heuristic) versiyonudur.

Her düğüm (n) için toplam maliyet fonksiyonu:
$$
f(n) = g(n) + h(n)
$$

- $g(n)$: Başlangıç düğümünden n düğümüne gelmenin gerçek maliyeti.
- $h(n)$: n düğümünden hedefe tahmini maliyet (Heuristic).

### Heuristic Fonksiyonları
Öklid Mesafesi (Euclidean):
$$
h(n) = \sqrt{(x_n - x_{hedef})^2 + (y_n - y_{hedef})^2}
$$

Manhattan Mesafesi (Izgara hareketi için):
$$
h(n) = |x_n - x_{hedef}| + |y_n - y_{hedef}|
$$

## 3. RRT (Rapidly-exploring Random Tree)
Yüksek boyutlu uzaylarda veya karmaşık engellerde rastgele örnekleme ile yol bulur.

**Algoritma Adımları:**
1. $T$ ağacını başlangıç noktası $q_{start}$ ile başlat.
2. Rastgele bir nokta $q_{rand}$ seç.
3. Ağaçta $q_{rand}$'a en yakın düğümü bul ($q_{near}$).
4. $q_{near}$'dan $q_{rand}$ yönüne doğru bir adım ($\Delta q$) ilerle ve yeni nokta $q_{new}$ oluştur.
5. Eğer $q_{near} \to q_{new}$ yolu engelsiz ise, $q_{new}$'i ağaca ekle.
6. Hedefe ulaşana kadar tekrarla.

RRT* (RRT-Star), bulunan yolu sürekli optimize ederek (rewiring) asimptotik olarak optimal çözüme yaklaşır.

## 4. Rota Düzgünleştirme (Path Smoothing)
A* veya RRT tarafından üretilen yollar genellikle köşelidir. Robotun akıcı hareketi için düzgünleştirilmelidir.

### Bezier Eğrileri veya B-Spline
Kontrol noktaları ($P_0, P_1, \dots, P_n$) kullanılarak parametrik eğri oluşturulur.
İkinci dereceden (Quadratic) Bezier Eğrisi:
$$
B(t) = (1-t)^2 P_0 + 2(1-t)t P_1 + t^2 P_2, \quad t \in [0, 1]
$$

Bu sayede keskin dönüşler yumuşatılarak robotun kinematik yapısına uygun hale getirilir.
