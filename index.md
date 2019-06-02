# Web Crawler

##### Apa itu *Web Crawler* ?

​	Web crawler merupakan suatu alat atau program yang digunakan search 
engine untuk meng index atau menjelajahi seluruh web yang ada di 
internet. Web craler juga biasa disebut web spider.

**Bagaimana cara kerja *Web Crawler* ?**
	Web crawler menggali setiap data yang ada di internet seperti seperti : 
meta data, keyword, dan lain sebagainya. Kemudian web crawler atau si 
(spider man) ini akan meng index seluruh data kita ke dalam data base 
search engine. Sampai pada akhirnya halaman website akan ditampilkan di 
SERP (search engine rage page).

##### Apa fungsi *Web Crawler* ?

1. Web crawler biasa
   digunakan untuk membuat salinan sebagian atau keseluruhan halaman web
   yang telah dikunjunginya agar dapat diproses lebih lanjut oleh system pengindeksan.
2. Web crawler dapat digunakan untuk proses pemeliharaan sebuah website, seperti memvalidasi kode html sebuah web.
3. Web crawler juga digunakan untuk memperoleh data yang khusus, seperti mengumpulkan alamat email.

##### Langkah - langkah yang harus dilakukan :

#### Tahap 1: Penginstallan dan Crawling Data

1. Install python versi 3.5

   Pada project ini menggunakan bahasa pemograman Python.

2. Install library yang digunakan yaitu *request dan beautifulsoup*  pada cmd folder Python yang sudah terinstall. Kemudian koneksikan library yang sudah di install tadi dengan cara import.

   ```
   ### untuk proses crawl data
   import requests
   from bs4 import BeautifulSoup
   ### untuk database
   import sqlite3
   import csv
   ```

3. Fungsi berikutnya yaitu mengubah data ke file csv agar lebih mudah diakses.

   ```
   def write_csv(nama_file, isi, tipe='w'):
       'tipe=w; write; tipe=a; append;'
       with open(nama_file, mode=tipe) as tbl:
           tbl_writer = csv.writer(tbl, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)
           for row in isi:
               tbl_writer.writerow(row)
   ```

4. Tentukan halaman web yang akan di crawling, disini menggunakan detik.com yang merupakan halaman web yang berisi berita online terupdate. Data yang akan Crawl yaitu judul dan isi dari berita terbaru. Lalu hubungkan dengan kode berikut :

   ```
   src = 'http://www.detik.com/'
   ```

   ![](C:\Users\ulvia ayu cahyana\AppData\Roaming\Typora\typora-user-images\1556784604785.png)

5. Selanjutnya fungsi requests untuk mengambil data dari halaman web

   ```
   def crawl(src):
       global c
       try :
           page = requests.get(src)
   ```

   Mengubah file html ke object beautiful soup 

   ```
   soup = BeautifulSoup(page.content, 'html.parser')
   ```

   Mengambil semua tautan di halaman

   ```
   links = soup.findAll(class_='desc_nhl')
           numPages = list()
   ```

   Selain di simpan di file csv mengakses pada setiap halaman lalu menyimpannya ke database. 

   ```
   for i in links:
               try :
                   print ('Proses : %.2f' %((c/150)*100) + '%'); c+=0.2
                   url = i.find ('a')['href']
                   print(url)
                   page = requests.get(url)
                   soup = BeautifulSoup(page.content, 'html.parser')
   
                   konten = soup.find(class_='detail_content')
   				### elemen judul
                   judul = konten.find('h1').getText()
                   print(judul)
   				### elemen isi berita
                   des = konten.find(class_= 'itp_bodycontent detail_text')
                   tmp = des.getText().replace('\n', ' ').replace('\t', ' ')
                   
   				### menyimpan ke database
                   conn.execute("INSERT INTO BERITA \
                                   VALUES (?, ?)", (judul, tmp));
               except AttributeError:
                   print("")
           conn.commit()pengecualian apabila ada nilai yang error
   except ValueError:
           print('Download selesai')
   ```

   Membuat Database baru untuk menyimpan data hasil crawling* dengan nama webMining.db

   ```
   conn = sqlite3.connect('webMining.db')
   c = 1
   choice = input("Update data? Y/N").lower()
   if choice == 'y':
       conn.execute('drop table if exists BERITA')
       conn.execute('''CREATE TABLE BERITA
                    (JUDUL     TEXT     NOT NULL,
                    DESKRIPSI  TEXT     NOT NULL);''')
       crawl(src)
       conn.commit()Reprosesion
   ###memanggil database yang sudah dibuat
   print("Building VSM...")
   cursor = conn.execute("SELECT * from BERITA")
   cursor = cursor.fetchall()
   cursor = cursor[:10]
   pertama = True
   corpus = list()
   c=1
   for row in cursor:
       print ('Proses : %.2f' %((c/len(cursor))*100) + '%'); c+=1
       txt = row[1]
       cleaned = preprosesing(txt)
       corpus.append(cleaned)
   ```

   #### Tahap 2 : Preprocessing

   Pada  tahap ini

   1. Install sklearn sama seperti install pada tahap penginstalan sebelumnya. Lalu masukkan kode berikut untuk menghubungkannya.

      ```
      from sklearn.feature_extraction.text import CountVectorizer
      from sklearn.feature_extraction.text import TfidfVectorizer
      from sklearn.cluster import KMeans
      from sklearn.metrics import silhouette_samples, silhouette_score
      import numpy as np
      ```

   1. Install juga library untuk stemming.

      Stemming nantinya akan memetakan bentuk dari suatu kata menjadi bentuk kata dasarnya, misalkan :

      |   Kata    | Kata Dasar |
      | :-------: | :--------: |
      | menjemput |   jemput   |
      |  berlari  |    lari    |

       import kodenya :

      ```
      from Sastrawi.Stemmer.StemmerFactory import StemmerFactory
      from Sastrawi.StopWordRemover.StopWordRemoverFactory import StopWordRemoverFactory
      ```

   2. Tahap selanjutnya harus menghilangkan kata tidak penting, seperti kata-kata imbuhan. berikut kodenya :

   ```
   def preprosesing(txt):
       SWfactory = StopWordRemoverFactory()
           stopword = SWfactory.create_stop_word_remover()
           stop = stopword.remove(txt)
   ```

   Berikut adalah proses stemming/mencari kata dasar dari kata yang sudah di temukan.  Kata yang mempunyai kata tidak penting (imbuhan) akan di filter menjadi kata dasar. 

   ```
   Sfactory = StemmerFactory()
       stemmer = Sfactory.create_stemmer()
   
       stem = stemmer.stem(stop)
       return stem
   ```

   #### Proses VSM

   Manfaat VSM secara umum adalah membantu memperbaiki proses secara
   menyeluruh dan meningkatkan efisiensi dan efektifitas proses.

   Fungsi berikut digunakan untuk menghitung banyaknya kata dalam satu string

   ```
   def countWord(txt):
        d = dict()
           for i  in txt.split():
               if d.get(i) == None:
                   d[i] = txt.count(i)
           return d
   ```

   Sedangkan fungsi ini digunakan untuk membangun VSM.

   ```
   def add_row_VSM(d):
       #init baris baru
       VSM.append([])
       # memasukkan kata berdasarkan kata yang telah ditemukan sebelumnya
       for i in VSM[0]:
           if d.get(i) == None:
               VSM[-1].append(0)
           else :
               VSM[-1].append(d.pop(i));
   
       # memasukkan kata baru 
       for i in d:
           VSM[0].append(i)
           for j in range(1, len(VSM)-1):
               VSM[j].insert(-2,0)
           VSM[-1].append(d.get(i))
   ```

   ##### TF - IDF

   ​	VSM dapat diaplikasikan dalam klasifikasi dokumen, clustering dokumen, 
   dan scoring dokumen terhadap sebuah query. Dalam VSM setiap dokumen 
   direpresentasikan sebagai sebuah vector, dimana nilai dari setiap nilai 
   dari vector tersebut mewakili weight sebuah term. Dalam menghitung 
   term-weight dapa digunakan beberapa teknik seperti frequency, 
   binary-document vector, atau tf-idf. Dalam postingan ini saya akan 
   menggunakan tf-idf dalam menghitung term-weighting sebuah dokumen. 

   ​	TF merupakan frekuensi kemunculan suatu term pada sebuah dokumen, sedangkan DF merupakan jumlah dokumen dimana terdapat term yang bersangkutan. 

   Berikut kodenya :

   ```
   ###proses perhitungan 
   vectorizer = TfidfVectorizer()
   tfidf_matrix = vectorizer.fit_transform(corpus)
   feature_name = vectorizer.get_feature_names()
   
   ###proses pemanggilan 
   write_csv("tfidf_1.csv", [feature_name])
   write_csv("tfidf_1.csv", tfidf_matrix.toarray(), 'a')
   ```

   Memasukkan data ke csv

   ```
   vectorizer = CountVectorizer(min_df=1, ngram_range=(1,1))
   BoW_matrix = vectorizer.fit_transform(corpus)
   write_csv("bows_lib.csv", [vectorizer.get_feature_names()])
   write_csv("bows_lib.csv", BoW_matrix.toarray())
   ```

   ##### Selection Fitur

   Pada proses ini akan menyeleksi fitur-fitur kata yang hanya dibutukan saja.

   ```
   def pearsonCalculate(data, u,v):
       "i, j is an index"
       atas=0; bawah_kiri=0; bawah_kanan = 0
       for k in range(len(data)):
           atas += (data[k,u] - meanFitur[u]) * (data[k,v] - meanFitur[v])
           bawah_kiri += (data[k,u] - meanFitur[u])**2
           bawah_kanan += (data[k,v] - meanFitur[v])**2
       bawah_kiri = bawah_kiri ** 0.5
       bawah_kanan = bawah_kanan ** 0.5
       return atas/(bawah_kiri * bawah_kanan)
   def meanF(data):
       meanFitur=[]
       for i in range(len(data[0])):
           meanFitur.append(sum(data[:,i])/len(data))
       return np.array(meanFitur)
   def seleksiFiturPearson(data, threshold):
       global meanFitur
       meanFitur = meanF(data)
       u=0
       while u < len(data[0]):
           dataBaru=data[:, :u+1]
           meanBaru=meanFitur[:u+1]
           v = u
           while v < len(data[0]):
               if u != v:
                   value = pearsonCalculate(data, u,v)
                   if value < threshold:
                       dataBaru = np.hstack((dataBaru, data[:, v].reshape(data.shape[0],1)))
                       meanBaru = np.hstack((meanBaru, meanFitur[v]))
                       kataBaru = np.hstack((kataBaru, katadasar[v]))
               v+=1
           data = dataBaru
           meanFitur = meanBaru
           katadasar = kataBaru
           if u%50 == 0 : print("Seleksi : ", u, data.shape)
           u+=1
       return data, kataBaru
   ```

   Proses pemanggilan *Selection Fitur*

   Pada proses ini yang dimana nantinya data selanjutnya akan disimpan ke file csv

   ```
   batas=0.8
   fiturBaru = seleksiFiturPearson(feature_name,tfidf_matrix.toarray(), batas)
   write_csv("selection_fitur.csv", [katabaru])
   write_csv("selection_fitur.csv", fiturBaru, 'a')
   ```

   ##### Claustering

   Pada claustering ini menggunakan fungsi kmeans

   K-Means sendiri adalah suatu metode penganalisaan data atau metode Data Mining 
   yang melakukan proses pemodelan tanpa supervisi (unsupervised) dan 
   merupakan salah satu metode yang melakukan pengelompokan data dengan 
   sistem partisi. berikut kodenya :

   ```
   kmeans = KMeans(n_clusters=5, random_state=0).fit(fiturBaru)
   write_csv("Kluster_label.csv", [kmeans.labels_])
   s_avg = silhouette_score(fiturBaru, kmeans.labels_, random_state=10)
   print(s_avg)
   for i in range(len(kmeans.labels_)):
       print("Doc %d =>> cluster %d" %(i+1, kmeans.labels_[i]))
   ```

#### Referensi

https://tehnikbagas.wordpress.com/2017/02/09/pengertian-web-crawler-dan-fungsinya/
https://liyantanto.wordpress.com/2011/06/28/stemming-bahasa-indonesia-dengan-algoritma-nazief-dan-andriani/
https://medium.com/python-pandemonium/develop-your-first-web-crawler-in-python-scrapy-6b2ee4baf954
https://www.crummy.com/software/BeautifulSoup/bs4/doc/
http://taufiksutanto.blogspot.com/2017/11/text-preprocessing-terms-distribution.html
https://belajarpython.com/2018/05/sastrawi-natural-language-processing-bahasa-indonesia.html
https://yudiagusta.wordpress.com/k-means/