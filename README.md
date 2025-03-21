# Commit 1 Reflection Notes

Pada commit ini, saya telah menambahkan fitur untuk menangani koneksi dari browser. Berikut adalah poin-poin penting dan refleksi dari pengerjaan:

1. **Membuat Listener dan Menangani Koneksi**  
   - Menggunakan `TcpListener::bind("127.0.0.1:7878")` untuk membuat server yang mendengarkan koneksi pada alamat dan port tersebut.  
   - Mengiterasi setiap koneksi yang masuk menggunakan `listener.incoming()`, sehingga setiap koneksi baru diproses satu per satu.

2. **Penggunaan Fungsi `handle_connection`**  
   - Fungsi `handle_connection` bertugas untuk membaca dan memproses data yang dikirim oleh browser.
   - Menggunakan `BufReader` untuk membungkus `TcpStream` sehingga memudahkan pembacaan baris demi baris.
   - Metode `.lines()` digunakan untuk membaca setiap baris yang dikirim. Fungsi `take_while(|line| !line.is_empty())` memastikan bahwa pembacaan berhenti ketika mencapai baris kosong, yang menandai akhir dari header HTTP.
   - Data permintaan HTTP (HTTP request) disimpan dalam sebuah `Vec` dan kemudian dicetak ke console untuk verifikasi.

3. **Pentingnya Pencetakan Permintaan**  
   - Dengan mencetak request, kita dapat melihat bagaimana browser berkomunikasi dengan server. Contoh output menunjukkan baris-baris header seperti `GET / HTTP/1.1`, `Host`, `User-Agent`, dan lain-lain.
   - Hal ini membantu dalam debugging dan memastikan bahwa server menerima serta membaca permintaan dengan benar.

4. **Refleksi Penggunaan Rust**  
   - Penggunaan Rust untuk membuat server sederhana menunjukkan kekuatan bahasa ini dalam menangani operasi I/O secara efisien.
   - Error handling dilakukan dengan cara yang sederhana menggunakan `unwrap()`. Meski ini baik untuk tahap awal pengembangan, nantinya perlu ditangani dengan cara yang lebih robust untuk produksi.
   - Pendekatan single-threaded ini cocok untuk eksperimen awal dan pembelajaran, namun perlu dikembangkan lebih lanjut (misalnya dengan multi-threading) untuk menangani beban yang lebih berat.

5. **Catatan Pengembangan Selanjutnya**  
   - Mengoptimalkan error handling dan menambahkan fitur pengiriman respon ke browser.
   - Mempertimbangkan penggunaan thread pool untuk mengelola banyak koneksi secara bersamaan.
   - Memperdalam pemahaman terkait protokol HTTP dan bagaimana server dapat merespon dengan berbagai status code dan konten yang sesuai.

Commit message yang digunakan: “(1) Handle-connection, check response”

# Commit 2 Reflection Notes

Pada commit ini, saya menambahkan kemampuan untuk mengembalikan halaman HTML sehingga browser dapat menampilkan konten. Berikut adalah beberapa poin refleksi mengenai perubahan yang dilakukan:

1. **Membaca File HTML**  
   - Saya menggunakan fungsi `fs::read_to_string("hello.html")` untuk membaca isi file HTML. Dengan cara ini, saya dapat dengan mudah memuat konten HTML yang ingin ditampilkan di browser.
   - Pendekatan ini membuat pemisahan antara logika server dan konten statis, sehingga jika ingin mengubah tampilan, cukup melakukan perubahan pada file `hello.html`.

2. **Menyusun HTTP Response**  
   - Saya membuat string response yang mencakup status line `"HTTP/1.1 200 OK"`, header `Content-Length` (yang berfungsi memberi tahu browser panjang konten yang dikirim), dan konten HTML itu sendiri.
   - Format response mengikuti protokol HTTP dengan menyertakan `\r\n` sebagai pemisah antara header dan body. Hal ini penting agar browser dapat mengenali batas antara header dan konten.

3. **Pengiriman Data ke Browser**  
   - Setelah string response selesai disusun, data dikirim ke browser menggunakan `stream.write_all(response.as_bytes())`.
   - Dengan cara ini, ketika browser melakukan request, ia akan menerima respons yang lengkap dan dapat menampilkan halaman HTML dengan benar.

4. **Pembelajaran Protokol HTTP**  
   - Proses ini membantu saya memahami lebih dalam tentang bagaimana komunikasi antara browser dan server terjadi melalui protokol HTTP.
   - Saya belajar tentang peran header seperti `Content-Length` dalam memastikan bahwa browser tahu seberapa banyak data yang akan diterima sehingga dapat memproses konten dengan benar.

5. **Implementasi Sederhana yang Efektif**  
   - Meskipun masih sederhana, implementasi ini merupakan fondasi yang kuat untuk pengembangan server yang lebih kompleks. Langkah selanjutnya adalah menangani berbagai jenis permintaan dan mengirimkan respon yang sesuai.
   - Hal ini juga menekankan pentingnya pemisahan antara logika pemrosesan permintaan dan penyajian konten, yang merupakan prinsip dasar dalam pengembangan web.

![commit 2 Response Screenshot](/assets/milestone%202.PNG)
<!-- ![Server Response Screenshot](assets\milestone 2.PNG) -->

# Commit 3 Reflection Notes

## Validating Request and Selectively Responding

Pada milestone ketiga ini, saya telah mengimplementasikan fitur validasi request dan memberikan respons yang sesuai berdasarkan path yang diminta. Berikut adalah refleksi dari proses pengembangan ini:

### 1. Proses Validasi Request

Dalam implementasi sebelumnya, server akan selalu mengembalikan file `hello.html` tanpa memperhatikan path yang diminta oleh klien. Sekarang, server melakukan validasi terhadap request line untuk menentukan respons yang sesuai:

```rust
let request_line = buf_reader.lines().next().unwrap().unwrap();

let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

Kode di atas memeriksa request line pertama dari HTTP request. Jika request tersebut adalah "GET / HTTP/1.1" (mengakses root path), maka server akan mengirimkan respons 200 OK dengan konten dari file `hello.html`. Namun, jika request line tidak cocok dengan pola tersebut (misalnya saat mengakses URL `/bad`), server akan mengirimkan respons 404 NOT FOUND dengan konten dari file `404.html`.

### 2. Pemisahan Respons dan Kebutuhan Refactoring

Refactoring kode diperlukan untuk memperbaiki struktur dan meningkatkan maintainability. Dalam implementasi sebelumnya, kode untuk menyiapkan dan mengirim respons HTTP terkumpul dalam satu blok. Dengan pemisahan respons, kita dapat:

1. **Meningkatkan Keterbacaan**: Dengan memisahkan logika untuk menentukan status dan file yang akan digunakan, kode menjadi lebih mudah dibaca dan dipahami.

2. **Fleksibilitas**: Pemisahan ini memungkinkan kita untuk dengan mudah menambahkan lebih banyak jenis respons di masa depan (misalnya 500 Internal Server Error, 301 Moved Permanently, dll).

3. **Single Responsibility Principle**: Setiap bagian kode memiliki tanggung jawab yang jelas - satu bagian menentukan jenis respons, bagian lain membaca file, dan bagian terakhir mengirimkan respons.

Perhatikan penggunaan `match` statement yang lebih ekspresif dan idiomatik dalam Rust dibandingkan dengan rangkaian if-else:

```rust
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

Pattern matching ini memungkinkan kita untuk dengan mudah memperluas fitur server untuk menangani berbagai jenis request di masa depan.

### 3. Hasil Pengujian

Setelah implementasi, server berhasil memberikan respons yang berbeda berdasarkan path yang diakses:

- Mengakses `http://127.0.0.1:7878/` menampilkan halaman selamat datang (hello.html)
- Mengakses `http://127.0.0.1:7878/bad` atau path lain yang tidak terdaftar menampilkan halaman 404 error (404.html)

![Server Response Screenshot](assets/milestone%203.PNG)
### 4. Kesimpulan

Implementasi validasi request dan respons selektif ini adalah langkah penting dalam pengembangan web server yang fungsional. Meskipun masih sederhana (hanya membedakan antara root path dan path lainnya), implementasi ini memberikan fondasi bagi fitur routing yang lebih kompleks di masa depan.

Refactoring yang dilakukan juga sangat penting untuk memastikan kode tetap maintainable seiring dengan bertambahnya kompleksitas. Dengan struktur kode yang bersih dan terorganisir, penambahan fitur baru akan lebih mudah dilakukan.


# Commit 4 Reflection Notes

## Simulation of Slow Requests

Pada commit ini, saya telah menambahkan simulasi untuk mendemonstrasikan keterbatasan server single-threaded ketika menghadapi permintaan yang membutuhkan waktu lama untuk diproses. Berikut adalah refleksi dari pengalaman ini:

### 1. Implementasi Endpoint yang Lambat

Saya menambahkan endpoint baru `/sleep` yang secara sengaja menunda respons selama 10 detik menggunakan `thread::sleep(Duration::from_secs(10))`:

```rust
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    "GET /sleep HTTP/1.1" => {
        thread::sleep(Duration::from_secs(10));
        ("HTTP/1.1 200 OK", "hello.html")
    }
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

Endpoint ini mensimulasikan operasi yang membutuhkan waktu lama seperti query database yang kompleks, komputasi yang berat, atau request ke layanan eksternal yang lambat.

### 2. Dampak pada Performa Server

Setelah melakukan pengujian dengan membuka dua jendela browser secara bersamaan, saya mengamati beberapa hal penting:

- **Blocking Behavior**: Ketika saya mengakses `127.0.0.1:7878/sleep` di satu jendela, kemudian mencoba mengakses `127.0.0.1:7878` di jendela lain, permintaan kedua harus menunggu hingga permintaan pertama selesai diproses (sekitar 10 detik).
  
- **Antrian Permintaan**: Server memproses permintaan secara sekuensial, sehingga semua permintaan setelah permintaan yang lambat akan mengalami penundaan, bahkan jika permintaan tersebut sangat sederhana dan seharusnya bisa diproses dengan cepat.

- **Pengalaman Pengguna yang Buruk**: Dalam skenario dunia nyata, pengguna yang mengakses endpoint cepat akan merasakan keterlambatan yang signifikan jika ada pengguna lain yang secara bersamaan mengakses endpoint lambat.

### 3. Akar Masalah: Single-Threaded Server

Masalah ini muncul karena implementasi server saat ini menggunakan pendekatan single-threaded:

```rust
for stream in listener.incoming() {
    let stream = stream.unwrap();
    handle_connection(stream);
}
```

Dalam model ini:
- Server hanya dapat menangani satu permintaan pada satu waktu
- Permintaan diproses secara sekuensial (satu setelah yang lain)
- Permintaan yang membutuhkan waktu lama akan memblokir semua permintaan berikutnya

### 4. Implikasi untuk Pengembangan Selanjutnya

Simulasi ini dengan jelas menunjukkan kebutuhan untuk solusi yang lebih baik untuk menangani beberapa koneksi secara bersamaan. Beberapa pendekatan yang dapat dipertimbangkan:

- **Multi-threading**: Membuat thread baru untuk setiap koneksi sehingga permintaan dapat diproses secara paralel
- **Thread Pool**: Membuat kumpulan thread yang dapat digunakan kembali untuk menangani permintaan, menghindari overhead pembuatan thread untuk setiap koneksi
- **Asynchronous I/O**: Menggunakan pendekatan non-blocking dengan async/await untuk menangani banyak koneksi secara efisien

### 5. Kesimpulan

Eksperimen ini memberikan pemahaman yang jelas tentang keterbatasan model single-threaded dalam pengembangan server web. Meskipun implementasi ini sederhana dan mudah dipahami, ia tidak cocok untuk lingkungan produksi di mana server harus menangani banyak permintaan secara bersamaan dan beberapa di antaranya mungkin membutuhkan waktu lama untuk diproses.

Pengalaman ini memperkuat pentingnya concurrency dalam desain server web dan menyiapkan dasar untuk milestone berikutnya di mana kita akan mengimplementasikan model multi-threaded atau thread pool untuk meningkatkan performa dan responsivitas server.

# Commit 5 Reflection Notes

Pada commit ini, saya mengimplementasikan server multithreaded dengan menggunakan thread pool sesuai dengan referensi [Turning Our Single-Threaded Server into a Multithreaded Server](https://rust-book.cs.brown.edu/ch21-02-multithreaded.html#turning-our-single-threaded-server-into-a-multithreaded-server). Berikut poin-poin utama dari implementasi ini:

1. **Peningkatan Responsivitas Server**
   - Dengan penggunaan thread pool, server kini dapat menangani beberapa koneksi secara paralel. Misalnya, koneksi yang menuju endpoint `/sleep` tidak akan memblokir request lain.
   - Server lebih responsif, dan pengalaman pengguna meningkat karena koneksi tidak terblokir meskipun ada request yang lambat.

2. **Implementasi Thread Pool**
   - Saya membuat thread pool dengan jumlah thread tetap (misalnya 4 thread) sehingga setiap koneksi masuk dialokasikan ke salah satu thread pool.
   - Pendekatan ini mengurangi overhead yang timbul dari pembuatan thread baru untuk setiap koneksi dan lebih efisien dalam penggunaan resource.

3. **Pembelajaran Mengenai Concurrency**
   - Implementasi ini memberikan pemahaman mendalam mengenai konsep concurrency dan bagaimana Rust mengelola thread melalui mekanisme ownership dan message passing.
   - Mengikuti contoh di referensi, saya belajar cara mengintegrasikan channel (`mpsc`) dan mutex untuk sinkronisasi antar thread.

# Commit Bonus Reflection Notes

Pada commit bonus ini, saya menambahkan fungsi `build` pada modul thread pool sebagai alternatif dari fungsi `new`. Peningkatan ini memiliki beberapa tujuan utama:

1. **Validasi Parameter Ukuran Thread Pool**
   - Fungsi `build` memastikan bahwa nilai ukuran thread pool harus lebih dari 0. Jika tidak, fungsi ini mengembalikan error dengan pesan yang menjelaskan bahwa thread count harus merupakan bilangan positif non-nol.
   - Hal ini meningkatkan keamanan dan robustitas aplikasi, karena mencegah pembuatan thread pool dengan parameter yang tidak valid.

2. **Penggunaan Error Handling**
   - Dengan mengembalikan `Result<ThreadPool, PoolCreationError>`, kita dapat menangani kemungkinan error secara lebih eksplisit dan mencegah crash secara tidak terduga.
   - Tipe error `PoolCreationError` telah diimplementasikan untuk mendukung trait `Display` dan `Debug`, sehingga memudahkan dalam proses debugging serta pelaporan error.

3. **Pemisahan Fungsi Pembuatan**
   - Fungsi `new` tetap ada sebagai alternatif yang tidak melakukan validasi, namun fungsi `build` memberikan lapisan proteksi ekstra.
   - Pendekatan ini memungkinkan fleksibilitas dalam pembuatan thread pool, sekaligus memberikan panduan kepada developer untuk menggunakan metode yang lebih aman.