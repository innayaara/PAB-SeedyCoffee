# ☕ PANDUAN UJIAN LENGKAP — SeedyCoffee
### Mata Kuliah: Pemrograman Aplikasi Bergerak

---

## 1. RINGKASAN PROJECT

**SeedyCoffee (SeedyOrder)** adalah aplikasi mobile pemesanan kopi berbasis Flutter dengan sistem **3 role pengguna**: Customer, Admin, dan Kasir.

- Customer dapat browse menu, pesan, dan bayar via kode unik/QR Code
- Admin dapat kelola menu/banner, lihat pesanan, dan analisis bisnis via AI
- Kasir dapat verifikasi pembayaran dengan scan QR atau input kode manual
- Backend: **Supabase** (Auth + PostgreSQL + Storage)
- AI: **Google Gemini 2.5 Flash** untuk insight dashboard admin

**Versi:** 3.0.0 | **Platform:** Android | **Tim:** 5 orang

---

## 2. FRAMEWORK & PACKAGE

**Framework:** Flutter (Dart), SDK ≥3.4.0

| Package | Fungsi |
|---|---|
| `provider ^6.1.2` | State management (ChangeNotifier) |
| `supabase_flutter ^2.8.4` | Backend: Auth, Database, Storage |
| `shared_preferences ^2.3.3` | Simpan session & cart lokal di HP |
| `google_fonts ^6.2.1` | Font Playfair Display + DM Sans |
| `qr_flutter ^4.1.0` | Generate QR Code dari kode pesanan |
| `mobile_scanner ^5.2.3` | Scan QR Code via kamera (kasir) |
| `fl_chart ^0.70.2` | Grafik batang pendapatan (dashboard) |
| `image_picker ^1.1.2` | Ambil foto dari galeri HP |
| `crop_your_image ^2.0.0` | Crop gambar sebelum upload |
| `intl ^0.19.0` | Format Rupiah & tanggal Indonesia |
| `http ^1.2.2` | HTTP request ke Gemini AI API |
| `uuid ^4.5.1` | Generate ID unik |
| `url_launcher ^6.3.1` | Buka WhatsApp dari app |
| `smooth_page_indicator ^1.2.1` | Animasi titik banner slider |

---

## 3. STRUKTUR FOLDER `lib/`

```
lib/
├── main.dart                    ← Entry point, routing, Provider setup
├── core/
│   ├── config/
│   │   ├── env_config.dart      ← Baca .env (API keys, feature flags)
│   │   └── supabase_config.dart ← Init koneksi Supabase
│   ├── constants/
│   │   ├── app_colors.dart      ← Semua warna (hitam, coklat kopi)
│   │   ├── app_constants.dart   ← Nama route & key SharedPreferences
│   │   └── app_text_styles.dart ← Semua style teks
│   ├── theme/app_theme.dart     ← ThemeData global MaterialApp
│   └── utils/helpers.dart       ← Format harga, generate kode unik
├── models/
│   ├── user_model.dart          ← Data user + enum UserRole
│   ├── menu_model.dart          ← Data menu + CategoryModel
│   ├── cart_item_model.dart     ← Item di keranjang
│   ├── order_model.dart         ← Pesanan + OrderItemModel + enum OrderStatus
│   ├── banner_model.dart        ← Banner promo
│   └── notification_model.dart  ← Notifikasi + enum NotificationType
├── providers/
│   └── app_provider.dart        ← PUSAT semua state (1 file)
├── services/
│   ├── auth_service.dart        ← Login, register OTP, logout, session
│   ├── menu_service.dart        ← CRUD menu & kategori
│   ├── order_service.dart       ← CRUD pesanan & konfirmasi bayar
│   ├── banner_service.dart      ← CRUD banner
│   ├── notification_service.dart← Buat & baca notifikasi
│   ├── storage_service.dart     ← Upload gambar ke Supabase Storage
│   ├── analytics_service.dart   ← Hitung statistik penjualan
│   ├── ai_service.dart          ← Integrasi Google Gemini AI
│   ├── whatsapp_service.dart    ← Broadcast WA via Fonnte API
│   └── static_database.dart    ← Data dummy (fallback tanpa internet)
├── screens/
│   ├── shared/splash_screen.dart         ← Loading + routing role
│   ├── auth/login_screen.dart            ← Login & register + OTP
│   ├── user/main_screen.dart             ← Bottom nav 4 tab (customer)
│   ├── user/home_tab.dart                ← Banner slider + grid menu
│   ├── user/menu_detail_screen.dart      ← Detail menu + pilih opsi
│   ├── user/cart_tab.dart                ← Keranjang belanja
│   ├── user/checkout_screen.dart         ← Checkout + tampil QR
│   ├── user/order_detail_screen.dart     ← Detail riwayat pesanan
│   ├── user/notification_tab.dart        ← Notifikasi
│   ├── user/profile_tab.dart             ← Edit profil & avatar
│   ├── admin/admin_screen.dart           ← Tab wrapper admin
│   ├── admin/admin_dashboard_tab.dart    ← Statistik + AI insight
│   ├── admin/admin_menu_tab.dart         ← List menu
│   ├── admin/admin_menu_form_screen.dart ← Form tambah/edit menu
│   ├── admin/admin_banner_tab.dart       ← Kelola banner
│   ├── admin/admin_orders_tab.dart       ← Semua pesanan
│   ├── admin/admin_promo_tab.dart        ← Kirim promo WA
│   └── kasir/kasir_screen.dart           ← Scan QR + konfirmasi bayar
└── widgets/
    ├── brew_button.dart         ← Tombol custom (primary/outline/ghost)
    ├── brew_text_field.dart     ← TextField custom dengan validasi
    ├── brew_snackbar.dart       ← Snackbar custom ikon & warna
    ├── menu_card.dart           ← Kartu menu di grid
    └── option_pill.dart         ← Chip selector size/sugar/ice
```

---

## 4. ALUR APLIKASI DARI `main.dart`

```
main() dipanggil
  ↓ init Flutter, format tanggal ID, cek .env, kunci portrait
  ↓ runApp → ChangeNotifierProvider → AppProvider.initialize()
  ↓ MaterialApp → initialRoute: '/' → SplashScreen

SplashScreen:
  ↓ animasi logo + progress bar (~2 detik)
  ↓ AppProvider memuat: menu, banner, users, session (paralel)
  ↓ cek role currentUser:
     → null/customer  → /main    (MainScreen)
     → admin          → /admin   (AdminScreen)
     → cashier        → /kasir   (KasirScreen)
     → tidak login    → /login   (LoginScreen)
```

**Daftar Route (`app_constants.dart`):**

| Route | Halaman |
|---|---|
| `/` | SplashScreen |
| `/login` | LoginScreen |
| `/main` | MainScreen (customer) |
| `/admin` | AdminScreen |
| `/kasir` | KasirScreen |
| `/checkout` | CheckoutScreen |

---

## 5. ALUR CUSTOMER: MENU → CART → CHECKOUT → QR → KASIR

```
[HomeTab] Grid menu → tap kartu
  ↓
[MenuDetailScreen]
  Lihat gambar, nama, deskripsi, harga
  Pilih: Size (S/M/L) | Gula (Normal/Less/No) | Es (Normal/Less/No)
  Atur jumlah | Tambah catatan
  Tap "Tambah ke Keranjang" → AppProvider.addToCart()
  ↓
[CartTab]
  Lihat semua item, ubah qty (+/-), hapus item
  Lihat total harga
  Tap "Checkout" → navigate ke /checkout
  ↓
[CheckoutScreen]
  Lihat ringkasan pesanan + total
  Tap "Konfirmasi & Checkout" → dialog konfirmasi
  → AppProvider.checkout()
    → OrderService.createOrder() → insert ke Supabase
    → Cart dikosongkan
    → Notifikasi dibuat
  ↓ SUKSES:
  Tampil QR Code (qr_flutter) dari kode pesanan
  Tampil kode unik (contoh: SEEDY-AB12) + total
  Tap "Lihat Riwayat" → kembali ke MainScreen
  ↓
[KasirScreen]
  Kasir scan QR / input kode manual
  → AppProvider.confirmPayment()
    → update status order: pending → confirmed
    → insert ke tabel payments (method: cash)
    → notifikasi ke customer: "Pesanan dikonfirmasi"
```

---

## 6. STATE MANAGEMENT — PROVIDER

**Pola yang digunakan:** `ChangeNotifier` + `Provider`

```dart
// Di main.dart — inject provider ke seluruh app
ChangeNotifierProvider(
  create: (_) => AppProvider()..initialize(),
  child: MaterialApp(...)
)

// Di widget — baca + subscribe perubahan
final p = context.watch<AppProvider>();  // rebuild otomatis

// Di fungsi tombol — baca tanpa subscribe
final p = context.read<AppProvider>();   // tidak rebuild
```

**Data dalam AppProvider (`lib/providers/app_provider.dart`):**

| Variabel state | Isi |
|---|---|
| `_currentUser` | User yang login (null jika belum login) |
| `_menus` | Semua menu kopi dari Supabase |
| `_categories` | Kategori menu |
| `_banners` | Banner promo |
| `_cart` | Item keranjang (disimpan ke SharedPreferences) |
| `_orders` | Pesanan (semua jika admin/kasir, milik sendiri jika customer) |
| `_notifications` | Notifikasi milik user aktif |
| `_isLoading` | Status loading |

**Getter penting:**
- `isLoggedIn` → apakah ada user login
- `cartItemCount` → total item di cart
- `cartTotal` → total harga cart
- `userOrders` → pesanan milik user aktif
- `unreadNotifCount` → jumlah notif belum dibaca

---

## 7. MODEL DATA PENTING

### UserModel
```dart
enum UserRole { customer, admin, cashier }

class UserModel {
  final String id;        // ID dari Supabase Auth
  final String fullName;  // Nama lengkap
  final UserRole role;    // Role: customer/admin/cashier
  final String? phone;    // Nomor HP
  final String? avatarUrl;// URL foto profil
  final String? email;    // Email
  final DateTime createdAt;
}
```

### MenuModel (field penting)
```dart
class MenuModel {
  final String id, name, categoryId, categoryName;
  final int price;
  final int? originalPrice;  // ada → berarti sedang diskon
  final bool isAvailable;    // false → tampil overlay "Sold Out"
  final String? badge;       // "Recommended", "New Menu", dll
  final List<String>? sizeOptions, sugarOptions, iceOptions;
}
```

### CartItemModel (field penting)
```dart
class CartItemModel {
  final String menuItemId, menuName;
  final String? size, sugar, ice; // pilihan kustomisasi
  final String notes;             // catatan tambahan
  final int quantity, unitPrice;
  int get subtotal => unitPrice * quantity; // harga total item
}
```

### OrderModel (field penting)
```dart
enum OrderStatus { pending, confirmed, cancelled }

class OrderModel {
  final String id, userId, orderCode; // orderCode = kode unik (SEEDY-AB12)
  final OrderStatus status;
  final int totalAmount;
  final String? confirmedBy; // ID kasir
  final DateTime? confirmedAt;
  final List<OrderItemModel> items;
  bool get isPaid => status == OrderStatus.confirmed;
}
```

---

## 8. SUPABASE & CRUD

**Tabel di Supabase:**

| Tabel | Dikelola oleh |
|---|---|
| `profiles` | AuthService |
| `categories` | MenuService |
| `menu_items` | MenuService |
| `orders` | OrderService |
| `order_items` | OrderService |
| `payments` | OrderService |
| `promo_banners` | BannerService |
| `notifications` | NotificationService |

**CREATE — Buat pesanan baru:**
```dart
// order_service.dart → createOrder()
// 1. Insert header order
await client.from('orders').insert({
  'user_id': userId, 'order_code': code,
  'status': 'pending', 'total_amount': total
}).select().single();

// 2. Insert setiap item
await client.from('order_items').insert({
  'order_id': orderId, 'menu_item_id': item.menuItemId,
  'quantity': item.quantity, 'unit_price': item.unitPrice
});
```

**READ — Baca pesanan dengan join:**
```dart
// order_service.dart → loadOrders()
await client
  .from('orders')
  .select('*, order_items(*, menu_items(name, image_url))')
  .eq('user_id', userId)
  .order('created_at', ascending: false);
```

**UPDATE — Konfirmasi pembayaran:**
```dart
// order_service.dart → confirmPayment()
await client.from('orders').update({
  'status': 'confirmed',
  'confirmed_by': cashierId,
  'confirmed_at': DateTime.now().toIso8601String()
}).eq('id', orderId);
```

**DELETE — Hapus menu:**
```dart
// menu_service.dart → deleteMenu()
await client.from('menu_items').delete().eq('id', menuId);
```

**Mode Fallback:** Jika `.env` tidak dikonfigurasi → pakai `SharedPreferences` → pakai `StaticDatabase` (data dummy hardcoded di `static_database.dart`)

---

## 9. PEMBAYARAN / ORDER / CHECKOUT

> **Pembayaran adalah SIMULASI — bukan payment gateway online.**

Alur lengkap:
```
Customer checkout → order di DB: status PENDING
  ↓
Tampil QR Code + kode unik ke customer
  ↓
Customer tunjukkan ke kasir (layar HP)
  ↓
Kasir: scan QR (mobile_scanner) ATAU input kode manual
  ↓
Kasir tap "Konfirmasi Pembayaran"
  ↓
DB: order status → CONFIRMED
DB: insert record ke tabel payments (method: cash)
  ↓
Customer terima notifikasi: "Pesanan #SEEDY-AB12 telah dikonfirmasi"
```

**File yang terlibat:**
- `checkout_screen.dart` → UI ringkasan + generate QR
- `kasir_screen.dart` → scan + konfirmasi
- `order_service.dart` → `createOrder()` dan `confirmPayment()`
- `app_provider.dart` → `checkout()` dan `confirmPayment()`

**Catatan:** Ada referensi Midtrans di `env_config.dart` tapi **belum diimplementasikan** di kode.

---

## 10. WIDGET CUSTOM & UI

| Widget | File | Fungsi |
|---|---|---|
| `BrewButton` | `brew_button.dart` | Tombol 3 style: **primary** (hitam solid), **outline** (border hitam), **ghost** (transparan) |
| `BrewTextField` | `brew_text_field.dart` | Input text dengan label, border rounded, validasi |
| `BrewSnackbar` | `brew_snackbar.dart` | Notifikasi bawah layar, warna hijau (sukses) / merah (error) |
| `MenuCard` | `menu_card.dart` | Kartu menu di grid: gambar, nama, harga, badge, overlay "Sold Out" |
| `OptionPill` | `option_pill.dart` | Chip selector untuk size/sugar/ice di halaman detail menu |
| `ImageCropScreen` | `image_crop_screen.dart` | Halaman crop gambar (dipanggil saat upload foto menu/banner/avatar) |

**Widget Flutter bawaan yang penting diketahui:**

| Widget | Digunakan untuk |
|---|---|
| `IndexedStack` | Bottom navigation (tab tidak rebuild saat ganti) |
| `SliverGrid` | Grid menu di home yang bisa scroll bersama header |
| `PageView` | Slider banner promo |
| `SmoothPageIndicator` | Titik animasi di banner |
| `QrImageView` | Tampilkan QR Code |
| `MobileScanner` | Kamera scan QR |
| `BarChart` (fl_chart) | Grafik penjualan di dashboard |
| `AnimatedContainer` | Animasi chip kategori aktif |
| `RefreshIndicator` | Pull-to-refresh di riwayat pesanan |

**Warna utama:** Monokrom (hitam `#0A0A0A`, putih, abu) + aksen coklat kopi `#7B4A1E`

---

## 11. DEPLOYMENT APK

**Command build:**
```bash
# Install dependencies
flutter pub get

# Build APK release
flutter build apk --dart-define-from-file=.env --release --split-per-abi
```

**Yang wajib disiapkan sebelum build:**
1. Buat file `.env` dari template `.env.example`
2. Isi `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `GEMINI_API_KEY`
3. Jalankan `flutter pub run flutter_launcher_icons` (jika icon belum di-generate)
4. Pastikan `minSdkVersion` ≥ 21 di `android/app/build.gradle`

**Hasil APK:** `build/app/outputs/flutter-apk/app-release.apk`

**Flag penjelasan:**
- `--dart-define-from-file=.env` → inject API key saat build
- `--split-per-abi` → APK terpisah per arsitektur (lebih kecil)
- `--release` → mode production (tidak ada debug overlay)

---

## 12. SCRIPT PENJELASAN 1 MENIT

> *"SeedyCoffee adalah aplikasi pemesanan kopi berbasis Flutter yang kami buat sebagai proyek akhir mata kuliah PAB. Aplikasi ini punya tiga role: Customer bisa browse menu, pilih kustomisasi kopi, checkout, dan bayar lewat QR code. Admin bisa kelola menu, banner, pantau penjualan, dan dapat analisis bisnis otomatis dari Google Gemini AI. Kasir bisa scan QR atau input kode untuk konfirmasi pembayaran. Backend-nya pakai Supabase untuk autentikasi, database, dan penyimpanan gambar. State management-nya pakai Provider. Fitur unggulannya ada tiga: QR Code payment, AI dashboard pakai Gemini, dan sistem multi-role yang lengkap."*

---

## 13. SCRIPT PENJELASAN TEKNIS 3 MENIT

> *"Secara teknis, project ini dibangun dengan Flutter menggunakan Dart, target platform Android. Untuk state management, kami menggunakan package Provider dengan pola ChangeNotifier. Semua state aplikasi terpusat di satu file yaitu AppProvider, yang menyimpan data user, menu, cart, orders, dan notifications. Ketika data berubah, dipanggil notifyListeners() sehingga widget yang menggunakan context.watch() otomatis terupdate.*
>
> *Backend menggunakan Supabase, yang menyediakan PostgreSQL database, autentikasi dengan OTP email, dan storage untuk gambar. Kami membuat service layer terpisah untuk setiap entitas: AuthService, MenuService, OrderService, dan seterusnya. Setiap service bisa fallback ke SharedPreferences atau data statis jika Supabase tidak dikonfigurasi.*
>
> *Alur utama aplikasi: user login dengan OTP email via Supabase Auth, kemudian sistem routing di SplashScreen mendeteksi role dan mengarahkan ke halaman yang sesuai. Customer bisa browse menu dengan filter kategori, pilih kustomisasi pesanan, checkout, dan mendapatkan QR code unik. Kasir bisa scan QR menggunakan package mobile_scanner untuk konfirmasi pembayaran secara real-time.*
>
> *Fitur tambahan: Admin punya dashboard analitik dengan grafik fl_chart dan integrasi Google Gemini 2.5 Flash untuk AI insight bisnis. Upload gambar menggunakan image_picker dan crop_your_image sebelum disimpan ke Supabase Storage. Build APK menggunakan flutter build apk dengan flag dart-define-from-file untuk inject environment variables."*

---

## 14. PERTANYAAN UJIAN & JAWABAN

**Q: Framework apa yang digunakan?**
A: Flutter (Dart), target Android, versi SDK ≥3.4.0.

**Q: State management apa yang digunakan?**
A: Provider. Semua state di `AppProvider` yang extends `ChangeNotifier`. Saat data berubah, dipanggil `notifyListeners()`.

**Q: Apa bedanya `context.watch` dan `context.read`?**
A: `watch` → widget subscribe, rebuild otomatis saat data berubah. `read` → baca sekali, tidak rebuild (dipakai di fungsi tombol).

**Q: Bagaimana sistem multi-role bekerja?**
A: `UserModel` punya field `role` (enum: customer/admin/cashier). Di `SplashScreen`, setelah AppProvider selesai init, cek role lalu navigate ke halaman yang sesuai.

**Q: Apa itu Supabase?**
A: Backend-as-a-service berbasis PostgreSQL open source. Di project ini dipakai untuk Auth (login/OTP), Database, dan Storage (gambar).

**Q: Bagaimana OTP registrasi bekerja?**
A: User daftar → Supabase Auth kirim OTP 6 digit ke email → user input OTP di app → `AuthService.verifyOtp()` → akun terverifikasi → masuk ke app.

**Q: Bagaimana session login disimpan?**
A: Setelah login, `UserModel` disimpan ke `SharedPreferences` dalam format JSON. Saat app dibuka lagi, `restoreSession()` membacanya dan verifikasi token Supabase masih aktif.

**Q: Apakah pembayaran nyata atau simulasi?**
A: Simulasi (cash di kasir). Tidak ada payment gateway online aktif. Customer dapat QR/kode → tunjukkan ke kasir → kasir konfirmasi → status order jadi `confirmed`.

**Q: Bagaimana cart disimpan?**
A: Di `SharedPreferences` sebagai JSON. Disimpan tiap kali cart berubah (`_saveCart()`), dimuat ulang saat app start (`_loadCart()`).

**Q: Package apa untuk QR Code?**
A: Generate QR: `qr_flutter`. Scan QR (kamera): `mobile_scanner`.

**Q: Apa fungsi Google Gemini di app ini?**
A: AI Insight di dashboard admin. Data penjualan dikirim ke Gemini API via HTTP, Gemini membalas analisis bisnis (ringkasan, highlights, rekomendasi, prediksi) dalam Bahasa Indonesia.

**Q: Apa itu StaticDatabase?**
A: File `static_database.dart` berisi data dummy hardcoded (menu, user, banner). Dipakai sebagai fallback terakhir jika Supabase tidak dikonfigurasi dan SharedPreferences kosong.

**Q: Apa itu IndexedStack pada MainScreen?**
A: Widget yang menampilkan satu child dari beberapa, tapi semua child tetap hidup di memory. Dipakai di bottom navigation agar tab tidak di-rebuild saat berpindah tab.

**Q: Package apa untuk grafik?**
A: `fl_chart ^0.70.2`. Dipakai untuk `BarChart` pendapatan 7 hari di dashboard admin.

**Q: Jelaskan alur checkout sampai konfirmasi!**
A: Customer pilih menu → tambah ke cart → checkout → order dibuat di Supabase (status: pending) → QR + kode ditampilkan → tunjukkan ke kasir → kasir scan/input → konfirmasi → status jadi confirmed → customer terima notifikasi.

**Q: Apa fungsi EnvConfig?**
A: Membaca variabel dari file `.env` (Supabase URL, API key Gemini, dll) yang diinject saat build dengan flag `--dart-define-from-file=.env`. Juga berisi feature flags seperti `useSupabase`, `useGemini`.

**Q: Bagaimana upload gambar bekerja?**
A: `image_picker` → ambil dari galeri → `crop_your_image` → crop sesuai rasio (1:1 menu, 8:3 banner) → `StorageService.uploadImage()` → upload ke Supabase Storage → dapat URL publik → simpan ke tabel.

---

## 15. YANG WAJIB SAYA HAFAL SEBELUM UJIAN

**Framework & tools:**
- Flutter + Dart, backend Supabase, state management Provider
- Versi: Flutter 3.x, SDK ≥3.4.0

**File paling penting:**
- `main.dart` → entry point, routing, setup Provider
- `app_provider.dart` → semua state & logika bisnis
- `splash_screen.dart` → routing berdasarkan role
- `order_service.dart` → CRUD pesanan & konfirmasi bayar
- `auth_service.dart` → login, register, OTP, session
- `checkout_screen.dart` → checkout + generate QR
- `kasir_screen.dart` → scan QR + konfirmasi bayar

**3 role user:**
- Customer: browse → cart → checkout → QR
- Admin: CRUD menu/banner, dashboard AI, promo WA
- Kasir: scan QR atau input kode → konfirmasi bayar

**Konsep kunci Provider:**
- `ChangeNotifier` + `notifyListeners()` = sinyal "data berubah"
- `context.watch` = subscribe (rebuild) | `context.read` = baca sekali
- Semua state ada di `AppProvider` (1 file)

**Database Supabase:**
- Tabel utama: `profiles`, `menu_items`, `orders`, `order_items`, `payments`, `notifications`
- CRUD: insert/select/update/delete via `SupabaseConfig.client.from('tabel')`
- Fallback: SharedPreferences → StaticDatabase

**Pembayaran:**
- SIMULASI (bukan gateway online)
- Flow: pending → (kasir konfirmasi) → confirmed
- QR dibuat dari kode pesanan (contoh: `SEEDY-AB12`)

**Widget custom:** BrewButton, BrewTextField, BrewSnackbar, MenuCard, OptionPill

**Build APK:**
```bash
flutter build apk --dart-define-from-file=.env --release --split-per-abi
```

**Package wajib hafal beserta fungsinya:**
- `provider` → state management
- `supabase_flutter` → backend
- `qr_flutter` → generate QR
- `mobile_scanner` → scan QR (kamera)
- `fl_chart` → grafik dashboard
- `http` → request ke Gemini AI
- `shared_preferences` → simpan lokal
- `image_picker + crop_your_image` → upload foto

---
*SeedyCoffee — Proyek Akhir PAB | Made with ☕ and Flutter*
