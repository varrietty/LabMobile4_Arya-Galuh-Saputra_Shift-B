- **Nama** : Arya Galuh Saputra
- **NIM** : H1D022022
- **Shift Lama** : C
- **Shift Baru** : B

# Tugas Pertemuan 5 - Penjelasan Proses Aplikasi Toko Kita

## Daftar Isi
1. [Persiapan](#persiapan)
2. [Proses Login](#proses-login)
3. [Proses Registrasi](#proses-registrasi)
4. [Menampilkan Daftar Produk](#menampilkan-daftar-produk)
5. [Menambah Produk Baru](#menambah-produk-baru)
6. [Melihat Detail Produk](#melihat-detail-produk)
7. [Mengubah Produk](#mengubah-produk)
8. [Menghapus Produk](#menghapus-produk)
9. [Proses Logout](#proses-logout)

## Persiapan

1. Jalankan server Apache dan MySQL.
2. Jalankan program CodeIgniter 4 sebagai API dengan perintah:
   ```
   php spark serve --host 192.168.1.13
   ```
   (IP address menyesuaikan dengan IPv4 laptop/komputer Anda)

3. Jalankan aplikasi Flutter, dengan perintah berikut menggunakan Chrome:
   ```
   flutter run -d chrome --web-browser-flag "--disable-web-security"
   ```

## Proses Login

### a. Mengisi Form Login

![Screenshot form login](docs/login.png)

User diminta untuk memasukkan email dan password pada form login.

Kode:

```dart
Widget _emailTextField() {
  return TextFormField(
    decoration: const InputDecoration(labelText: "Email"),
    keyboardType: TextInputType.emailAddress,
    controller: _emailTextboxController,
    validator: (value) {
      if (value!.isEmpty) {
        return 'Email harus diisi';
      }
      return null;
    },
  );
}

Widget _passwordTextField() {
  return TextFormField(
    decoration: const InputDecoration(labelText: "Password"),
    keyboardType: TextInputType.text,
    obscureText: true,
    controller: _passwordTextboxController,
    validator: (value) {
      if (value!.isEmpty) {
        return "Password harus diisi";
      }
      return null;
    },
  );
}
```

### b. Proses Autentikasi

![Screenshot proses login](docs/proses_login.png)

Setelah menekan tombol login, aplikasi akan mengirim permintaan ke API untuk melakukan autentikasi.

Kode:

```dart
void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true;
  });

  LoginBloc.login(
          email: _emailTextboxController.text,
          password: _passwordTextboxController.text)
      .then((value) async {
    if (value.code == 200) {
      await UserInfo().setToken(value.token ?? "");
      await UserInfo().setUserID(int.tryParse(value.userID.toString()) ?? 0);

      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (BuildContext context) => SuccessDialog(
          description: "Login berhasil",
          okClick: () {
            Navigator.pushReplacement(
              context,
              MaterialPageRoute(
                builder: (context) => const ProdukPage(),
              ),
            );
          },
        ),
      );
    } else {
      showDialog(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) => const WarningDialog(
                description: "Login gagal, silahkan coba lagi",
              ));
    }
  }, onError: (error) {
    print(error);
    showDialog(
        context: context,
        barrierDismissible: false,
        builder: (BuildContext context) => const WarningDialog(
              description: "Login gagal, silahkan coba lagi",
            ));
  });

  setState(() {
    _isLoading = false;
  });
}
```

### c. Hasil Login

![Screenshot popup berhasil login](docs/sukses_login.png)
![Screenshot popup gagal login](docs/gagal_login.png)

Setelah proses autentikasi, user akan melihat popup yang menginformasikan hasil login (berhasil atau gagal).

## Proses Registrasi

### a. Mengisi Form Registrasi

![Screenshot form registrasi](docs/regist.png)

User diminta untuk mengisi nama, email, password, dan konfirmasi password.

Kode:

```dart
Widget _namaTextField() {
  return TextFormField(
    decoration: const InputDecoration(labelText: "Nama"),
    keyboardType: TextInputType.text,
    controller: _namaTextboxController,
    validator: (value) {
      if (value!.length < 3) {
        return "Nama harus diisi minimal 3 karakter";
      }
      return null;
    },
  );
}

// ... (kode untuk email, password, dan konfirmasi password)
```

### b. Proses Pengiriman Data Registrasi

![Screenshot proses registrasi](docs/proses_regist.png)

Setelah menekan tombol registrasi, aplikasi akan mengirim data ke API.

Kode:

```dart
void _submit() {
    _formKey.currentState!.save();
    setState(() {
      _isLoading = true;
    });
    RegistrasiBloc.registrasi(
      nama: _namaTextboxController.text,
      email: _emailTextboxController.text,
      password: _passwordTextboxController.text,
    ).then((value) {
      if (value['status']) {
        showDialog(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) => SuccessDialog(
            description: "Registrasi berhasil, silahkan login",
            okClick: () {
              Navigator.pop(context);
            },
          ),
        );
      } else {
        showDialog(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) => WarningDialog(
            description: value['message'],
          ),
        );
      }
    }).catchError((error) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (BuildContext context) => const WarningDialog(
          description: "Registrasi gagal, silahkan coba lagi",
        ),
      );
    }).whenComplete(() {
      setState(() {
        _isLoading = false;
      });
    });
  }
```

### c. Hasil Registrasi

![Screenshot popup berhasil registrasi](docs/sukses_regist.png)
![Screenshot popup gagal registrasi](docs/gagal_regist.png)

User akan melihat popup yang menginformasikan hasil registrasi.

## Menampilkan Daftar Produk

### a. Halaman Utama Produk

![Screenshot halaman daftar produk](docs/produk.png)

Setelah login berhasil, user akan diarahkan ke halaman utama yang menampilkan daftar produk.

Kode:

```dart
class ProdukPage extends StatefulWidget {
  const ProdukPage({Key? key}) : super(key: key);
  @override
  _ProdukPageState createState() => _ProdukPageState();
}

class _ProdukPageState extends State<ProdukPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('List Produk'),
        actions: [
          Padding(
              padding: const EdgeInsets.only(right: 20.0),
              child: GestureDetector(
                child: const Icon(Icons.add, size: 26.0),
                onTap: () async {
                  Navigator.push(context,
                      MaterialPageRoute(builder: (context) => ProdukForm()));
                },
              ))
        ],
      ),
      body: FutureBuilder<List>(
        future: ProdukBloc.getProduks(),
        builder: (context, snapshot) {
          if (snapshot.hasError) print(snapshot.error);
          return snapshot.hasData
              ? ListProduk(
                  list: snapshot.data,
                )
              : const Center(
                  child: CircularProgressIndicator(),
                );
        },
      ),
    );
  }
}
```

### b. Proses Pengambilan Data Produk

Data produk diambil dari API menggunakan `ProdukBloc.getProduks()`.

Kode:

```dart
class ProdukBloc {
  static Future<List<Produk>> getProduks() async {
    String apiUrl = ApiUrl.listProduk;
    var response = await Api().get(apiUrl);
    var jsonObj = json.decode(response.body);
    List<dynamic> listProduk = (jsonObj as Map<String, dynamic>)['data'];
    List<Produk> produks = [];
    for (int i = 0; i < listProduk.length; i++) {
      produks.add(Produk.fromJson(listProduk[i]));
    }
    return produks;
  }
}
```

## Menambah Produk Baru

### a. Membuka Form Tambah Produk

![Screenshot form tambah produk](docs/tambah_produk.png)

User dapat menambah produk baru dengan menekan ikon "+" di halaman utama.

### b. Mengisi Data Produk Baru

User diminta untuk mengisi kode produk, nama produk, dan harga produk.

Kode:

```dart
Widget _kodeProdukTextField() {
  return TextFormField(
    decoration: const InputDecoration(labelText: "Kode Produk"),
    keyboardType: TextInputType.text,
    controller: _kodeProdukTextboxController,
    validator: (value) {
      if (value!.isEmpty) {
        return "Kode Produk harus diisi";
      }
      return null;
    },
  );
}

// ... (kode untuk nama produk dan harga produk)
```

### c. Proses Penyimpanan Produk Baru

![Screenshot proses tambah produk](docs/proses_tambah_produk.png)

Setelah menekan tombol simpan, aplikasi akan mengirim data ke API.

Kode:

```dart
simpan() {
  setState(() {
    _isLoading = true;
  });
  Produk createProduk = Produk(id: null);
  createProduk.kodeProduk = _kodeProdukTextboxController.text;
  createProduk.namaProduk = _namaProdukTextboxController.text;
  createProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  ProdukBloc.addProduk(produk: createProduk).then((value) {
    if (value['status']) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (BuildContext context) => SuccessDialog(
          description: "Produk berhasil ditambahkan",
          okClick: () {
            Navigator.of(context).pushReplacement(
              MaterialPageRoute(
                builder: (BuildContext context) => const ProdukPage(),
              ),
            );
          },
        ),
      );
    } else {
      showDialog(
        context: context,
        builder: (BuildContext context) => WarningDialog(
          description: value['message'],
        ),
      );
    }
  }).catchError((error) {
    showDialog(
      context: context,
      builder: (BuildContext context) => const WarningDialog(
        description: "Simpan gagal, silahkan coba lagi",
      ),
    );
  }).whenComplete(() {
    setState(() {
      _isLoading = false;
    });
  });
}
```

### d. Hasil Penambahan Produk

![Screenshot popup berhasil tambah produk](docs/sukses_tambah.png)
![Screenshot popup gagal tambah produk](docs/gagal_tambah.png)

User akan melihat popup yang menginformasikan hasil penambahan produk.

## Melihat Detail Produk

### a. Memilih Produk dari Daftar

[Screenshot pemilihan produk dari daftar]

User dapat melihat detail produk dengan menekan item produk di daftar.

### b. Halaman Detail Produk

![Screenshot halaman detail produk](docs/detail_produk.png)

Halaman ini menampilkan informasi lengkap tentang produk yang dipilih.

Kode:

```dart
class ProdukDetail extends StatefulWidget {
  Produk? produk;
  ProdukDetail({Key? key, this.produk}) : super(key: key);
  @override
  _ProdukDetailState createState() => _ProdukDetailState();
}

class _ProdukDetailState extends State<ProdukDetail> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail Produk'),
      ),
      body: Center(
        child: Column(
          children: [
            Text(
              "Kode : ${widget.produk!.kodeProduk}",
              style: const TextStyle(fontSize: 20.0),
            ),
            Text(
              "Nama : ${widget.produk!.namaProduk}",
              style: const TextStyle(fontSize: 18.0),
            ),
            Text(
              "Harga : Rp. ${widget.produk!.hargaProduk.toString()}",
              style: const TextStyle(fontSize: 18.0),
            ),
            _tombolHapusEdit()
          ],
        ),
      ),
    );
  }

  // ...
}
```

## Mengubah Produk

### a. Membuka Form Ubah Produk

![Screenshot form ubah produk](docs/ubah.png)

Dari halaman detail produk, user dapat menekan tombol "EDIT" untuk membuka form ubah produk.

### b. Mengisi Perubahan Data Produk

Form ubah produk akan terisi dengan data produk yang ada, dan user dapat mengubahnya.

### c. Proses Penyimpanan Perubahan

![Screenshot proses ubah produk](docs/proses_ubah.png)

Setelah menekan tombol ubah, aplikasi akan mengirim data perubahan ke API.

Kode:

```dart
ubah() {
  setState(() {
    _isLoading = true;
  });
  Produk updateProduk = Produk(id: widget.produk!.id!);
  updateProduk.kodeProduk = _kodeProdukTextboxController.text;
  updateProduk.namaProduk = _namaProdukTextboxController.text;
  updateProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  ProdukBloc.updateProduk(produk: updateProduk).then((value) {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (BuildContext context) => SuccessDialog(
        description: "Produk berhasil diubah",
        okClick: () {
          Navigator.of(context).push(
            MaterialPageRoute(
              builder: (BuildContext context) => const ProdukPage(),
            ),
          );
        },
      ),
    );
  }, onError: (error) {
    showDialog(
        context: context,
        builder: (BuildContext context) => const WarningDialog(
              description: "Permintaan ubah data gagal, silahkan coba lagi",
            ));
  });
  setState(() {
    _isLoading = false;
  });
}
```

### d. Hasil Perubahan Produk

![Screenshot popup berhasil ubah produk](docs/sukses_ubah.png)

User akan melihat popup yang menginformasikan hasil perubahan produk.

## Menghapus Produk

### a. Memilih Produk untuk Dihapus

Dari halaman detail produk, user dapat menekan tombol "DELETE" untuk menghapus produk.

### b. Konfirmasi Penghapusan

![Screenshot dialog konfirmasi hapus](docs/dialog.png)

Sebelum menghapus, aplikasi akan menampilkan dialog konfirmasi.

Kode:

```dart
void confirmHapus() {
  AlertDialog alertDialog = AlertDialog(
    content: const Text("Yakin ingin menghapus data ini?"),
    actions: [
      OutlinedButton(
        child: const Text("Ya"),
        onPressed: () async {
          // Proses penghapusan
        },
      ),
      OutlinedButton(
        child: const Text("Batal"),
        onPressed: () => Navigator.pop(context),
      ),
    ],
  );
  showDialog(builder: (context) => alertDialog, context: context);
}
```

### c. Proses Penghapusan

Jika user mengkonfirmasi, aplikasi akan mengirim permintaan hapus ke API.

Kode:

```dart
onPressed: () async {
  bool success = await ProdukBloc.deleteProduk(
      id: int.parse(widget.produk!.id!));
  if (success) {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (BuildContext context) => SuccessDialog(
        description: "Produk berhasil dihapus",
        okClick: () {
          Navigator.of(context).pushReplacement(
            MaterialPageRoute(
              builder: (context) => const ProdukPage(),
            ),
          );
        },
      ),
    );
  } else {
    // Jika penghapusan gagal
    showDialog(
      context: context,
      builder: (BuildContext context) => const WarningDialog(
        description: "Hapus gagal, silahkan coba lagi",
      ),
    );
  }
},
```

### d. Hasil Penghapusan Produk

![Screenshot popup berhasil hapus produk](docs/sukses_hapus.png)

User akan melihat popup yang menginformasikan hasil penghapusan produk.

## Proses Logout

### a. Memilih Menu Logout

![Screenshot drawer dengan menu logout](docs/logout.png)

User dapat mengakses menu logout dari drawer aplikasi.

Kode:

```dart
Drawer(
  child: ListView(
    children: [
      ListTile(
        title: const Text('Logout'),
        trailing: const Icon(Icons.logout),
        onTap: () async {
          await LogoutBloc.logout().then((value) => {
                Navigator.of(context).pushAndRemoveUntil(
                    MaterialPageRoute(builder: (context) => LoginPage()),
                    (route) => false)
              });
        },
      )
    ],
  ),
),
```

### b. Proses Logout

Ketika user memilih logout, aplikasi akan menghapus token dan informasi user dari penyimpanan lokal.

Kode:

```dart
class LogoutBloc {
  static Future logout() async {
    await UserInfo().logout();
  }
}

// Di dalam class UserInfo
Future logout() async {
  final SharedPreferences pref = await SharedPreferences.getInstance();
  pref.clear();
}
```

### c. Kembali ke Halaman Login

Setelah proses logout selesai, user akan diarahkan kembali ke halaman login.