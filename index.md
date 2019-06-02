# WEB STRUCTURE MINING 

------

##### Pendahuluan

​	Tujuan <u>*Web Structure Mining*</u> adalah untuk menghasilkan ringkasan struktural tentang situs Web dan halaman Web.  Secara teknis, <u>*Web Content Mining*</u> terutama berfokus pada struktur dokumen dalam, sementara <u>*Web Structure Mining*</u> mencoba menemukan struktur tautan hyperlink di tingkat antar-dokumen.  Berdasarkan topologi hyperlink, <u>*Web Structure Mining*</u>  akan mengkatagorikan halaman web dan menghasilkan informasi, seperti kesamaan dan hubungan antara berbagai situs web.

​	Penambangan struktur web memiliki hubungan alami dengan <u>*Web Content Mining*</u> , karena sangat mungkin bahwa dokumen Web berisi tautan, dan keduanya menggunakan data nyata atau primer di Web.  Cukup sering menggabungkan kedua tugas mining ini dalam suatu aplikasi. Dalam setiap alamat web pasti akan mempunyai **PageRank.**



### Instalasi

Untuk menjalankan program kali ini pastikan laptop atau komputer Anda sudah terinstal python. 

Proses Crawl dengan menggunakan *BeautifulSoup* dapat diinstal dengan mengetik **pip install beautifulsoup4 dan pip install requests**.

```
import requests
from bs4 import BeautifulSoup
```

Sedangkan untuk membuat Graph menggunakan *matplotlib.pyplot*

```
import networkx as nx
import matplotlib.pyplot as plt
```

Terakhir instal library **pandas**

```
import pandas as pd
```

### Proses 

Sekarang Anda sudah menginstal library yang diperlukan, bisa langsung melanjutkan dengan langkah selanjutnya yaitu ketentuan link yang akan di Crawl nantinya.

Pada tahap ini akan mengecek ketentuan link yang diinginkan. 

```
def simplifiedURL(url):
    '''
    asumsi: alamat url tidak mengandung http(s) (misalnya "true-http-website.com"
            atau www (misalnya "true-www-website.com")
    '''
    # cek 1 : www
    if "www." in url:
        ind = url.index("www.")+4
        url = url[ind:]
    # cek 2 : http/https
    if not "http" in url:
        url = "http://"+url
    # cek 3 : tanda / di akhir
    if url[-1] == "/":
        url = url[:-1]
    return url
```

Selanjutnya tahap **Crawling link** disini format link yang lebih dari satu akan disamakan agar tidak ditampilkan berkali kali dan lebih efisien. Diproses ini kedalaman Crawling diatur dengan increment.

```
def crawl(url, max_deep,  show=False, deep=0):
    # returnnya ada di edgelist, 
    global edgelist
    
    # menyamakan format url, agar tidak ada url yg dobel
    url = simplifiedURL(url)
    
    # crawl semua link
    links = getAllLinks(url)
    
    # menambah counter kedalaman
    deep += 1

    #menampilkan proses
    if show:
        if deep == 1:
            print("(%d)%s" %(len(links),url))
        else:
            print("|", end="")
            for i in range(deep-1): print("--", end="")
            print("(%d)%s" %(len(links),url))
    
    for link in links:
        # Membentuk format jalan (edge => (dari, ke))
        edge = (url,link)
        # Mengecek jalan, apabila belum dicatat, maka dimasukkan ke list
        if not edge in edgelist:
            edgelist.append(edge)
        # Cek kedalaman, jika belum sampai terakhir, maka crawling.
        if (deep != max_deep):
            crawl(link, max_deep, show, deep)
```

Tidak semua link yang terdapat dalam website tersebut aktif, kadang kala ada link yang sudah tidak aktif lagi kemungkinan karena memang sudah tidak dipergunakan ataupun karena kendala lain. 

Berikut langkah untuk mencegahan eror apabila link yang diambil sudak tidak aktif/mati.

```
def getAllLinks(src):
    try:
        # Get page html
        ind = src.find(':')+3
        url = src[ind:]
        page = requests.get(src)

        # Mengubah html ke object beautiful soup
        soup = BeautifulSoup(page.content, 'html.parser')

        # GET all tag <a>
        tags = soup.findAll("a")

        links = []
        for tag in tags:
            # Pencegahan eror apabila link tidak memiliki href
            try:
                # Get all link
                link = tag['href']
                if not link in links and 'http' in link:
                    links.append(link)
            except KeyError:
                pass
        return links
    except:
        #print("Error 404 : Page "+src+" not found")
        return list()
```

Inisialisasi variabel awal untuk website. 

Disini mengambil dari website **detik.com**. 

```
root = "http://detik.com/"
nodelist = [root]
done = [root]
edgelist = []
```

### Menampilkan/Output

Menampilkan link yang sudah di Crawl dengan script sebagai berikut :

```
#crawl
tampilkan = True
crawl(root, 3, show=tampilkan)
if tampilkan:
    for i in range(10): print("")
```

Dalam script berikut berfungsi untuk **membuat Graph**

```
g = nx.from_pandas_edgelist(edgelistFrame, "From", "To", None, nx.DiGraph())

```

Mendeklarasikan pos (koordinat) 

```
pos = nx.spring_layout(g)

```

Menghitung pagerank dalam link website

##### Apa itu PageRank ?

​	PageRank adalah cara mengukur pentingnya halaman situs web. PageRank
bekerja dengan menghitung jumlah dan kualitas tautan ke suatu halaman 
untuk menentukan perkiraan kasar seberapa penting situs web itu.  Asumsi yang mendasarinya adalah bahwa situs web yang lebih penting cenderung menerima lebih banyak tautan dari situs web lain. Berikut contoh kerja PageRank :

![1557973495674](C:\Users\ulvia ayu cahyana\AppData\Roaming\Typora\typora-user-images\1557973495674.png)

> **PageRanks** Matematika untuk jaringan sederhana, dinyatakan dalam persentase.  (Google menggunakan skala logaritmik) Page C memiliki PageRank yang lebih tinggi daripada Page E, meskipun ada lebih sedikit tautan ke C;  satu tautan ke C berasal dari halaman penting dan karenanya bernilai tinggi. Jika peselancar web yang memulai pada halaman acak memiliki kemungkinan 85% untuk memilih tautan acak dari halaman yang sedang mereka kunjungi, dan kemungkinan 15% melompat ke halaman yang dipilih secara acak dari seluruh web, mereka akan mencapai Halaman E 8.1% dari waktu. (Kemungkinan 15% untuk melompat ke halaman sewenang-wenang sesuai dengan faktor redaman 85%.) Tanpa redaman, semua peselancar web akhirnya akan berakhir pada Halaman A, B, atau C, dan semua halaman lain akan memiliki PageRank nol. Di hadapan redaman, Halaman A secara efektif menautkan ke semua halaman di web, meskipun ia tidak memiliki tautan keluar sendiri.

Berikut tahapan scriptnya :

```
damping = 0.85
max_iterr = 100
error_toleransi = 0.0001
pr = nx.pagerank(g, alpha = damping, max_iter=max_iterr, tol=error_toleransi)

```

Selanjutnya yaitu membuat label dan menampilkan pagerank. Tampilan disini akan mengurutkan mulai dari pagerank terbesar ke pagerank terkecil.

```
print("keterangan node:")
nodelist = g.nodes
label= {}
data = []
for i, key in enumerate(nodelist):
    data.append((pr[key], key))
    label[key]=i
    #print(i, key, pr[key])

urut = data.copy()
for x in range(len(urut)):
    for y in range(len(urut)):
        if urut[x][0] > urut[y][0]:
            urut[x],urut[y] = urut[y],urut[x]
            
urut = pd.DataFrame(urut, None, ("PageRank", "Node"))
print(urut)

```

Menggambar Graph

```
nx.draw(g, pos)
nx.draw_networkx_labels(g, pos, label)

```

Pada tahapan terakhir yaitu menampilkan figure

```
plt.axis("off")
plt.show()

```



###### <u>**Referensi**</u>

<https://www.slideshare.net/AmirFahmideh/web-mining-structure-mining>

<https://translate.google.com/translate?hl=id&sl=en&u=https://www.quora.com/What-is-Web-Structure-Mining&prev=search>

<https://translate.google.com/translate?hl=id&sl=en&u=https://en.wikipedia.org/wiki/Web_mining&prev=search>

<https://id.wikipedia.org/wiki/PageRank>

<https://en.wikipedia.org/wiki/PageRank>

