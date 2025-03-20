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

