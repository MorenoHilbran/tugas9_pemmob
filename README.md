# TUGAS 9 - PERTEMUAN 11

```
Nama      : Moreno Hilbran Glenardi
NIM       : H1D023024
Shift     : B
Shift KRS : H
```

## Screenshot Aplikasi
<img width="385" height="781" alt="iPhone-13-PRO-localhost (1)" src="https://github.com/user-attachments/assets/8b345877-4c43-4661-9b6e-ad15801e9eec" />
<img width="385" height="781" alt="iPhone-13-PRO-localhost (2)" src="https://github.com/user-attachments/assets/4809daa9-d73c-4407-ba5d-ab6dc7742fd5" />
<img width="385" height="781" alt="iPhone-13-PRO-localhost (3)" src="https://github.com/user-attachments/assets/ec76435f-5e8d-48be-a96b-e5cbfa0d84c0" />
<img width="385" height="781" alt="iPhone-13-PRO-localhost (4)" src="https://github.com/user-attachments/assets/4756cd9e-606b-4981-9a5f-e5ea9619e62e" />
<img width="385" height="781" alt="iPhone-13-PRO-localhost (5)" src="https://github.com/user-attachments/assets/45b0eb7f-8979-4952-b321-1a916a59b838" />
<img width="385" height="781" alt="iPhone-13-PRO-localhost" src="https://github.com/user-attachments/assets/0d003c14-5f3c-49a1-8a41-878f71ab9f56" />

---

## 📱 PROSES LOGIN

### A. Form Login dan Input Data
Form login terdiri dari dua input field utama yaitu Email dan Password. User menginputkan kredensial mereka untuk mengakses aplikasi.

**Screenshot Form:**
- Form Login menampilkan logo toko "Berkah Abadi"
- Terdapat 2 TextFormField: Email dan Password
- Tombol Login dengan loading indicator
- Link navigasi ke halaman Registrasi

**Penjelasan Proses Input:**
1. User membuka aplikasi dan diarahkan ke `LoginPage`
2. User mengisi field Email (contoh: `moreno@gmail.com`)
3. User mengisi field Password 
4. Sistem melakukan validasi input:
   - Email tidak boleh kosong
   - Password tidak boleh kosong
5. Setelah validasi berhasil, user menekan tombol "Login"
6. Aplikasi menampilkan loading indicator selama proses autentikasi

**Kode Form Login** (`lib/ui/login_page.dart`):

```dart
// Widget Email TextField dengan validasi
Widget _emailTextField() {
  return TextFormField(
    decoration: InputDecoration(
      labelText: "Email",
      prefixIcon: const Icon(Icons.email),
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    ),
    keyboardType: TextInputType.emailAddress,
    controller: _emailTextboxController,
    validator: (value) {
      // Validasi harus diisi
      if (value!.isEmpty) {
        return 'Email harus diisi';
      }
      return null;
    },
  );
}

// Widget Password TextField dengan validasi
Widget _passwordTextField() {
  return TextFormField(
    decoration: InputDecoration(
      labelText: "Password",
      prefixIcon: const Icon(Icons.lock),
      border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    ),
    keyboardType: TextInputType.text,
    obscureText: true, // Menyembunyikan teks password
    controller: _passwordTextboxController,
    validator: (value) {
      if (value!.isEmpty) {
        return "Password harus diisi";
      }
      return null;
    },
  );
}

// Widget Tombol Login dengan loading indicator
Widget _buttonLogin() {
  return SizedBox(
    width: double.infinity,
    height: 48,
    child: ElevatedButton.icon(
      icon: _isLoading
          ? const SizedBox(
              width: 20,
              height: 20,
              child: CircularProgressIndicator(
                color: Colors.white,
                strokeWidth: 2,
              ),
            )
          : const Icon(Icons.login),
      label: Text(
        _isLoading ? "Memproses..." : "Login",
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
      onPressed: () {
        var validate = _formKey.currentState!.validate();
        if (validate) {
          if (!_isLoading) _submit();
        }
      },
    ),
  );
}
```

### B. Proses Autentikasi dan Popup

**Penjelasan Proses Login:**
1. Setelah form tervalidasi, method `_submit()` dipanggil
2. State `_isLoading` diset menjadi `true` untuk menampilkan loading indicator
3. `LoginBloc.login()` mengirim request ke API dengan email dan password
4. Response dari API berisi code status dan token
5. Jika berhasil (code 200):
   - Token disimpan menggunakan `UserInfo().setToken()`
   - User ID disimpan menggunakan `UserInfo().setUserID()`
   - User dinavigasi ke `ProdukPage` (halaman utama)
6. Jika gagal:
   - Menampilkan `WarningDialog` dengan pesan "Login gagal, silahkan coba lagi"

**Kode Proses Submit Login** (`lib/ui/login_page.dart`):

```dart
void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true; // Aktifkan loading indicator
  });
  
  // Panggil LoginBloc untuk proses autentikasi
  LoginBloc.login(
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text
  ).then((value) async {
    // Jika login berhasil (code 200)
    if (value.code == 200) {
      // Simpan token dan user ID ke local storage
      await UserInfo().setToken(value.token.toString());
      await UserInfo().setUserID(int.parse(value.userID.toString()));
      
      // Navigate ke halaman Produk
      Navigator.pushReplacement(
        context, 
        MaterialPageRoute(builder: (context) => const ProdukPage())
      );
    } else {
      // Jika login gagal, tampilkan dialog peringatan
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (BuildContext context) => const WarningDialog(
          description: "Login gagal, silahkan coba lagi",
        )
      );
    }
  }, onError: (error) {
    // Handle error dari API
    print(error);
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (BuildContext context) => const WarningDialog(
        description: "Login gagal, silahkan coba lagi",
      )
    );
  });
  
  setState(() {
    _isLoading = false; // Matikan loading indicator
  });
}
```

**Kode LoginBloc** (`lib/bloc/login_bloc.dart`):

```dart
class LoginBloc {
  static Future<Login> login({String? email, String? password}) async {
    String apiUrl = ApiUrl.login;
    var body = {"email": email, "password": password};
    
    // Kirim POST request ke API
    var response = await Api().post(apiUrl, body);
    var jsonObj = json.decode(response.body);
    
    // Convert response JSON ke object Login
    return Login.fromJson(jsonObj);
  }
}
```

**Kode Warning Dialog** (`lib/widget/warning_dialog.dart`):

```dart
class WarningDialog extends StatelessWidget {
  final String? description;
  final VoidCallback? okClick;

  const WarningDialog({Key? key, this.description, this.okClick})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Dialog(
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(Consts.padding)
      ),
      elevation: 0.0,
      backgroundColor: Colors.transparent,
      child: Container(
        padding: const EdgeInsets.all(Consts.padding),
        decoration: BoxDecoration(
          color: Colors.white,
          shape: BoxShape.rectangle,
          borderRadius: BorderRadius.circular(Consts.padding),
          boxShadow: const [
            BoxShadow(
              color: Colors.black26,
              blurRadius: 10.0,
              offset: Offset(0.0, 10.0),
            ),
          ],
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              "GAGAL",
              style: TextStyle(
                fontSize: 24.0, 
                fontWeight: FontWeight.w700, 
                color: Colors.red
              ),
            ),
            const SizedBox(height: 16.0),
            Text(
              description!,
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 16.0),
            ),
            const SizedBox(height: 24.0),
            Align(
              alignment: Alignment.bottomRight,
              child: ElevatedButton(
                onPressed: () {
                  Navigator.of(context).pop(); // Tutup dialog
                },
                child: const Text("OK"),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

**Screenshot Popup:**
- **Popup Berhasil**: User dinavigasi langsung ke `ProdukPage` tanpa popup sukses
- **Popup Gagal**: Dialog merah dengan tulisan "GAGAL" dan pesan "Login gagal, silahkan coba lagi"

---

## 📦 PROSES TAMBAH DATA PRODUK

### A. Form Tambah Produk

**Penjelasan:**
Form tambah produk dapat diakses dari halaman `ProdukPage` dengan menekan icon "+" di AppBar. Form ini terdiri dari 3 input field:
1. **Kode Produk** - Input teks untuk kode unik produk
2. **Nama Produk** - Input teks untuk nama produk
3. **Harga** - Input angka untuk harga produk

**Kode Form Tambah Produk** (`lib/ui/produk_form.dart`):

```dart
class ProdukForm extends StatefulWidget {
  Produk? produk;
  ProdukForm({Key? key, this.produk}) : super(key: key);
  
  @override
  _ProdukFormState createState() => _ProdukFormState();
}

class _ProdukFormState extends State<ProdukForm> {
  final _formKey = GlobalKey<FormState>();
  bool _isLoading = false;
  String judul = "TAMBAH PRODUK";
  String tombolSubmit = "SIMPAN";
  
  final _kodeProdukTextboxController = TextEditingController();
  final _namaProdukTextboxController = TextEditingController();
  final _hargaProdukTextboxController = TextEditingController();

  @override
  void initState() {
    super.initState();
    isUpdate(); // Cek apakah mode tambah atau edit
  }

  // Method untuk cek mode form (tambah atau edit)
  isUpdate() {
    if (widget.produk != null) {
      // Mode EDIT - isi form dengan data existing
      setState(() {
        judul = "UBAH PRODUK";
        tombolSubmit = "UBAH";
        _kodeProdukTextboxController.text = widget.produk!.kodeProduk!;
        _namaProdukTextboxController.text = widget.produk!.namaProduk!;
        _hargaProdukTextboxController.text = 
            widget.produk!.hargaProduk.toString();
      });
    } else {
      // Mode TAMBAH - form kosong
      judul = "TAMBAH PRODUK";
      tombolSubmit = "SIMPAN";
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(judul)),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.all(8.0),
          child: Form(
            key: _formKey,
            child: Column(
              children: [
                _kodeProdukTextField(),
                _namaProdukTextField(),
                _hargaProdukTextField(),
                _buttonSubmit()
              ],
            ),
          ),
        ),
      ),
    );
  }

  // Widget TextField Kode Produk dengan validasi
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

  // Widget TextField Nama Produk dengan validasi
  Widget _namaProdukTextField() {
    return TextFormField(
      decoration: const InputDecoration(labelText: "Nama Produk"),
      keyboardType: TextInputType.text,
      controller: _namaProdukTextboxController,
      validator: (value) {
        if (value!.isEmpty) {
          return "Nama Produk harus diisi";
        }
        return null;
      },
    );
  }

  // Widget TextField Harga dengan validasi
  Widget _hargaProdukTextField() {
    return TextFormField(
      decoration: const InputDecoration(labelText: "Harga"),
      keyboardType: TextInputType.number,
      controller: _hargaProdukTextboxController,
      validator: (value) {
        if (value!.isEmpty) {
          return "Harga harus diisi";
        }
        return null;
      },
    );
  }

  // Widget Tombol Submit (SIMPAN atau UBAH)
  Widget _buttonSubmit() {
    return OutlinedButton(
      child: Text(tombolSubmit),
      onPressed: () {
        var validate = _formKey.currentState!.validate();
        if (validate) {
          if (!_isLoading) {
            if (widget.produk != null) {
              ubah(); // Panggil method ubah jika mode edit
            } else {
              simpan(); // Panggil method simpan jika mode tambah
            }
          }
        }
      }
    );
  }
}
```

### B. Proses Simpan Data Produk

**Penjelasan Proses:**
1. User mengisi form (Kode Produk, Nama Produk, Harga)
2. User menekan tombol "SIMPAN"
3. Sistem validasi input - semua field harus diisi
4. Jika valid, state `_isLoading` diset `true`
5. Data dari form dimasukkan ke object `Produk`
6. `ProdukBloc.addProduk()` mengirim POST request ke API
7. Jika berhasil: User dinavigasi ke `ProdukPage` (list produk direfresh)
8. Jika gagal: Menampilkan `WarningDialog` dengan pesan error

**Kode Method Simpan** (`lib/ui/produk_form.dart`):

```dart
simpan() {
  setState(() {
    _isLoading = true; // Aktifkan loading
  });
  
  // Buat object Produk baru
  Produk createProduk = Produk(id: null);
  createProduk.kodeProduk = _kodeProdukTextboxController.text;
  createProduk.namaProduk = _namaProdukTextboxController.text;
  createProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  
  // Kirim data ke API melalui ProdukBloc
  ProdukBloc.addProduk(produk: createProduk).then((value) {
    // Jika berhasil, navigate ke ProdukPage
    Navigator.of(context).push(MaterialPageRoute(
      builder: (BuildContext context) => const ProdukPage()
    ));
  }, onError: (error) {
    // Jika gagal, tampilkan dialog peringatan
    showDialog(
      context: context,
      builder: (BuildContext context) => const WarningDialog(
        description: "Simpan gagal, silahkan coba lagi",
      )
    );
  });
  
  setState(() {
    _isLoading = false; // Matikan loading
  });
}
```

**Kode ProdukBloc untuk Tambah Data** (`lib/bloc/produk_bloc.dart`):

```dart
class ProdukBloc {
  static Future addProduk({Produk? produk}) async {
    String apiUrl = ApiUrl.createProduk;
    
    // Siapkan body request
    var body = {
      "kode_produk": produk!.kodeProduk,
      "nama_produk": produk.namaProduk,
      "harga": produk.hargaProduk.toString()
    };
    
    // Kirim POST request ke API
    var response = await Api().post(apiUrl, body);
    var jsonObj = json.decode(response.body);
    
    // Return status dari response
    return jsonObj['status'];
  }
}
```

---

## 🔄 PROSES UBAH/EDIT DATA PRODUK

**Penjelasan:**
1. User membuka detail produk dari list
2. User menekan tombol "EDIT" di halaman detail
3. Navigate ke `ProdukForm` dengan parameter produk existing
4. Form otomatis terisi dengan data produk yang dipilih
5. User mengubah data yang diinginkan
6. User menekan tombol "UBAH"
7. `ProdukBloc.updateProduk()` mengirim PUT request ke API
8. Jika berhasil: Navigate ke `ProdukPage` (data terupdate)
9. Jika gagal: Tampilkan `WarningDialog`

**Kode Method Ubah** (`lib/ui/produk_form.dart`):

```dart
ubah() {
  setState(() {
    _isLoading = true;
  });
  
  // Buat object Produk dengan ID existing
  Produk updateProduk = Produk(id: widget.produk!.id!);
  updateProduk.kodeProduk = _kodeProdukTextboxController.text;
  updateProduk.namaProduk = _namaProdukTextboxController.text;
  updateProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);
  
  // Kirim update request ke API
  ProdukBloc.updateProduk(produk: updateProduk).then((value) {
    Navigator.of(context).push(MaterialPageRoute(
      builder: (BuildContext context) => const ProdukPage()
    ));
  }, onError: (error) {
    showDialog(
      context: context,
      builder: (BuildContext context) => const WarningDialog(
        description: "Permintaan ubah data gagal, silahkan coba lagi",
      )
    );
  });
  
  setState(() {
    _isLoading = false;
  });
}
```

**Kode ProdukBloc untuk Update** (`lib/bloc/produk_bloc.dart`):

```dart
static Future updateProduk({required Produk produk}) async {
  String apiUrl = ApiUrl.updateProduk(int.parse(produk.id!));
  
  var body = {
    "kode_produk": produk.kodeProduk,
    "nama_produk": produk.namaProduk,
    "harga": produk.hargaProduk.toString()
  };
  
  // Kirim PUT request dengan body JSON
  var response = await Api().put(apiUrl, jsonEncode(body));
  var jsonObj = json.decode(response.body);
  
  return jsonObj['status'];
}
```

---

## 🗑️ PROSES HAPUS DATA PRODUK

**Penjelasan:**
1. User membuka detail produk
2. User menekan tombol "DELETE"
3. Muncul `AlertDialog` konfirmasi: "Yakin ingin menghapus data ini?"
4. Jika user menekan "Ya":
   - `ProdukBloc.deleteProduk()` mengirim DELETE request ke API
   - Jika berhasil: Navigate ke `ProdukPage` (produk terhapus dari list)
   - Jika gagal: Tampilkan `WarningDialog`
5. Jika user menekan "Batal": Dialog ditutup, tidak ada perubahan

**Kode Hapus Produk** (`lib/ui/produk_detail.dart`):

```dart
Widget _tombolHapusEdit() {
  return Row(
    mainAxisSize: MainAxisSize.min,
    children: [
      // Tombol Edit
      OutlinedButton(
        child: const Text("EDIT"),
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => ProdukForm(
                produk: widget.produk!,
              ),
            ),
          );
        },
      ),
      // Tombol Hapus
      OutlinedButton(
        child: const Text("DELETE"),
        onPressed: () => confirmHapus(),
      ),
    ],
  );
}

void confirmHapus() {
  AlertDialog alertDialog = AlertDialog(
    content: const Text("Yakin ingin menghapus data ini?"),
    actions: [
      // Tombol Ya - Hapus data
      OutlinedButton(
        child: const Text("Ya"),
        onPressed: () {
          ProdukBloc.deleteProduk(
            id: int.parse(widget.produk!.id!)
          ).then((value) => {
            // Jika berhasil, navigate ke ProdukPage
            Navigator.of(context).push(MaterialPageRoute(
              builder: (context) => const ProdukPage()
            ))
          }, onError: (error) {
            // Jika gagal, tampilkan warning dialog
            showDialog(
              context: context,
              builder: (BuildContext context) => const WarningDialog(
                description: "Hapus gagal, silahkan coba lagi",
              )
            );
          });
        },
      ),
      // Tombol Batal
      OutlinedButton(
        child: const Text("Batal"),
        onPressed: () => Navigator.pop(context),
      )
    ],
  );
  showDialog(builder: (context) => alertDialog, context: context);
}
```

**Kode ProdukBloc untuk Delete** (`lib/bloc/produk_bloc.dart`):

```dart
static Future<bool> deleteProduk({int? id}) async {
  String apiUrl = ApiUrl.deleteProduk(id!);
  
  // Kirim DELETE request ke API
  var response = await Api().delete(apiUrl);
  var jsonObj = json.decode(response.body);
  
  // Return data status dari response
  return (jsonObj as Map<String, dynamic>)['data'];
}
```

---

## 📋 PROSES LIHAT LIST & DETAIL PRODUK

### List Produk

**Penjelasan:**
- `ProdukPage` menggunakan `FutureBuilder` untuk fetch data produk dari API
- `ProdukBloc.getProduks()` mengirim GET request
- Data ditampilkan dalam `ListView` dengan `Card` widget
- Setiap item menampilkan nama dan harga produk
- Tap pada item untuk membuka detail produk

**Kode List Produk** (`lib/ui/produk_page.dart`):

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
          // Icon untuk tambah produk
          Padding(
            padding: const EdgeInsets.only(right: 20.0),
            child: GestureDetector(
              child: const Icon(Icons.add, size: 26.0),
              onTap: () async {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => ProdukForm()),
                );
              },
            ),
          )
        ],
      ),
      drawer: Drawer(
        child: ListView(
          children: [
            ListTile(
              title: const Text('Logout'),
              trailing: const Icon(Icons.logout),
              onTap: () async {
                await LogoutBloc.logout().then((value) => {
                  Navigator.of(context).pushAndRemoveUntil(
                    MaterialPageRoute(builder: (context) => const LoginPage()),
                    (route) => false
                  )
                });
              },
            )
          ],
        ),
      ),
      body: FutureBuilder<List>(
        future: ProdukBloc.getProduks(), // Fetch data produk
        builder: (context, snapshot) {
          if (snapshot.hasError) print(snapshot.error);
          return snapshot.hasData
              ? ListProduk(list: snapshot.data)
              : const Center(child: CircularProgressIndicator());
        },
      ),
    );
  }
}

class ListProduk extends StatelessWidget {
  final List? list;
  const ListProduk({Key? key, this.list}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: list == null ? 0 : list!.length,
      itemBuilder: (context, i) {
        return ItemProduk(produk: list![i]);
      },
    );
  }
}

class ItemProduk extends StatelessWidget {
  final Produk produk;
  const ItemProduk({Key? key, required this.produk}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        // Navigate ke detail produk saat item di-tap
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => ProdukDetail(produk: produk),
          ),
        );
      },
      child: Card(
        child: ListTile(
          title: Text(produk.namaProduk!),
          subtitle: Text(produk.hargaProduk.toString()),
        ),
      ),
    );
  }
}
```

### Detail Produk

**Penjelasan:**
- Halaman detail menampilkan informasi lengkap produk
- Menampilkan: Kode Produk, Nama Produk, dan Harga
- Tersedia tombol EDIT dan DELETE

**Kode Detail Produk** (`lib/ui/produk_detail.dart`):

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
}
```

---

## 👤 PROSES REGISTRASI

**Penjelasan:**
User yang belum memiliki akun dapat membuat akun baru melalui halaman registrasi.

### Form Registrasi

Form registrasi memiliki 4 input field:
1. **Nama** - Minimal 3 karakter
2. **Email** - Format email valid
3. **Password** - Minimal 6 karakter
4. **Konfirmasi Password** - Harus sama dengan password

**Kode Registrasi** (`lib/ui/registrasi_page.dart`):

```dart
Widget _buttonRegistrasi() {
  return SizedBox(
    width: double.infinity,
    height: 48,
    child: ElevatedButton.icon(
      icon: _isLoading
          ? const SizedBox(
              width: 20,
              height: 20,
              child: CircularProgressIndicator(
                color: Colors.white,
                strokeWidth: 2,
              ),
            )
          : const Icon(Icons.app_registration),
      label: Text(
        _isLoading ? "Memproses..." : "Registrasi",
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
      onPressed: () {
        var validate = _formKey.currentState!.validate();
        if (validate) {
          if (!_isLoading) _submit();
        }
      },
    ),
  );
}

void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true;
  });
  
  // Kirim data registrasi ke API
  RegistrasiBloc.registrasi(
    nama: _namaTextboxController.text,
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text
  ).then((value) {
    // Jika berhasil, tampilkan success dialog
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (BuildContext context) => SuccessDialog(
        description: "Registrasi berhasil, silahkan login",
        okClick: () {
          Navigator.pop(context); // Kembali ke halaman login
        },
      )
    );
  }, onError: (error) {
    // Jika gagal, tampilkan warning dialog
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (BuildContext context) => const WarningDialog(
        description: "Registrasi gagal, silahkan coba lagi",
      )
    );
  });
  
  setState(() {
    _isLoading = false;
  });
}
```

**Kode Success Dialog** (`lib/widget/success_dialog.dart`):

```dart
class SuccessDialog extends StatelessWidget {
  final String? description;
  final VoidCallback? okClick;

  const SuccessDialog({Key? key, this.description, this.okClick})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Dialog(
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(Consts.padding)
      ),
      elevation: 0.0,
      backgroundColor: Colors.transparent,
      child: Container(
        padding: const EdgeInsets.all(Consts.padding),
        decoration: BoxDecoration(
          color: Colors.white,
          shape: BoxShape.rectangle,
          borderRadius: BorderRadius.circular(Consts.padding),
          boxShadow: const [
            BoxShadow(
              color: Colors.black26,
              blurRadius: 10.0,
              offset: Offset(0.0, 10.0),
            ),
          ],
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              "SUKSES",
              style: TextStyle(
                fontSize: 24.0,
                fontWeight: FontWeight.w700,
                color: Colors.green
              ),
            ),
            const SizedBox(height: 16.0),
            Text(
              description!,
              textAlign: TextAlign.center,
              style: const TextStyle(fontSize: 16.0),
            ),
            const SizedBox(height: 24.0),
            Align(
              alignment: Alignment.bottomRight,
              child: OutlinedButton(
                onPressed: () {
                  Navigator.of(context).pop(); // Tutup dialog
                  okClick!(); // Callback action
                },
                child: const Text("OK"),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

---

## 🔐 PROSES LOGOUT

**Penjelasan:**
1. User membuka drawer menu di `ProdukPage`
2. User menekan menu "Logout"
3. `LogoutBloc.logout()` menghapus token dan user ID dari local storage
4. User dinavigasi kembali ke `LoginPage`
5. Semua halaman sebelumnya dihapus dari navigation stack

**Kode Logout** (di `lib/ui/produk_page.dart`):

```dart
drawer: Drawer(
  child: ListView(
    children: [
      ListTile(
        title: const Text('Logout'),
        trailing: const Icon(Icons.logout),
        onTap: () async {
          await LogoutBloc.logout().then((value) => {
            // Hapus semua halaman dan navigate ke LoginPage
            Navigator.of(context).pushAndRemoveUntil(
              MaterialPageRoute(builder: (context) => const LoginPage()),
              (route) => false // Hapus semua route sebelumnya
            )
          });
        },
      )
    ],
  ),
)
```

---

## 🌟 Fitur Utama

* **Autentikasi Pengguna:**
    * Halaman Login dengan validasi input
    * Halaman Registrasi dengan validasi form yang ketat (validasi email, password, dll)
    * Popup berhasil/gagal untuk feedback user
* **Manajemen Produk (CRUD):**
    * **List Produk:** Menampilkan daftar produk dengan desain kartu yang interaktif
    * **Detail Produk:** Melihat rincian informasi produk (Kode, Nama, Harga)
    * **Tambah Produk:** Form untuk menambah data produk baru
    * **Edit Produk:** Form untuk mengubah data produk existing
    * **Hapus Produk:** Konfirmasi dialog sebelum menghapus data
* **Navigasi:** Menggunakan *Drawer Menu* dan navigasi halaman standar Flutter (`Navigator`)
* **UI/UX:** Desain antarmuka modern dengan *Gradient Background* dan dialog feedback

## 📂 Struktur Kode

### 1. Konfigurasi Utama
* **`main.dart`**
    * Titik awal (*entry point*) aplikasi
    * Mengatur tema global aplikasi dengan warna dominan hijau
    * Mengarahkan pengguna ke `LoginPage`

### 2. Model Data (`/model`)
* **`produk.dart`** - Model data produk (ID, Kode, Nama, Harga)
* **`login.dart`** - Model response login (Token, User ID, Code, Status)
* **`registrasi.dart`**
    * Model untuk menangani respon status pendaftaran akun baru.

### 3. Antarmuka Otentikasi (`/ui`)
* **`login_page.dart`**
    * Halaman formulir login.
    * Memiliki validasi input email dan password.
    * Berpindah ke `ProdukPage` jika login berhasil, atau ke `RegistrasiPage` jika ingin mendaftar.
* **`registrasi_page.dart`**
    * Halaman pendaftaran pengguna baru.
    * Memiliki validasi ketat: Nama (min 3 karakter), Email (format regex), dan Password (min 6 karakter & konfirmasi cocok).

### 4. Antarmuka Produk (`/ui`)
* **`produk_page.dart`**
    * Dashboard utama yang menampilkan daftar produk.
    * Dilengkapi dengan **Sidebar (Drawer)** yang berisi profil admin ("Moreno") dan tombol Logout.
    * Menampilkan data produk menggunakan widget `ItemProduk`.
* **`produk_detail.dart`**
    * Halaman detail untuk melihat informasi spesifik satu produk.
    * Berisi tombol aksi untuk **Edit** (navigasi ke form) dan **Hapus** (memunculkan dialog konfirmasi).
* **`produk_form.dart`**
    * Halaman formulir yang bersifat dinamis (Reuasable).
    * **Mode Tambah:** Jika dibuka tanpa data, form kosong dan tombol berlabel "SIMPAN".
    * **Mode Edit:** Jika dibuka dengan membawa data produk, form terisi otomatis dan tombol berlabel "UBAH".
