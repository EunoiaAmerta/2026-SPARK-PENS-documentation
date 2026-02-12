# Panduan Pengembangan Backend SparkPens

**Versi**: 1.0 (Februari 2026)
**Target Pembaca**: Pemula / Mahasiswa
**Teknologi**: .NET 10, PostgreSQL, Entity Framework Core
Daftar isi ini mencakup perjalanan dari instalasi hingga API CRUD kamu berhasil berjalan.

## 1. Persiapan Lingkungan (Linux/Arch)

### 1.1 Instalasi .NET 10 SDK

Di Arch/EndeavourOS, jalankan perintah: `sudo pacman -S dotnet-sdk`

### 1.2 Instalasi PostgreSQL

sudo pacman -S postgresql
_Inisialisasi database (hanya sekali)_
sudo -u postgres initdb -D /var/lib/postgres/data
_jalankan service_
sudo systemctl enable --now postgresql

### 1.3 Fix Masalah Path (Penting!)

Jika kamu menggunakan Zsh, tambahkan jalur tools .NET ke profil kamu agar command `dotnet-ef` bisa dipanggil:
`echo 'export PATH="$PATH:$HOME/.dotnet/tools"' >> ~/.zshrc`
`source ~/.zshrc`

## 2. Inisialisasi Proyek

Kita membuat proyek Web API dari nol.
_Membuat folder proyek_
`mkdir 2026-SPARK-PENS-backend && cd 2026-SPARK-PENS-backend`
_Membuat Web API baru_
`dotnet new webapi -n SparkPens.Api`
`cd SparkPens.Api`
**_Troubleshooting: Error .NET 10_**
Jika saat instalasi package muncul error _NETSDK1226_, buka file _SparkPens.Api.csproj_ dan tambahkan baris ini di dalam <PropertyGroup>:
`<AllowMissingPrunePackageData>true</AllowMissingPrunePackageData>`

## 3. Instalasi "Senjata" (NuGet Packages)

Kita butuh beberapa pustaka tambahan agar .NET bisa bicara dengan database.
_Driver PostgreSQL untuk Entity Framework_
`dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL`
_Alat desain migrasi_
`dotnet add package Microsoft.EntityFrameworkCore.Design`
_UI Dokumentasi API (Swagger)_
`dotnet add package Swashbuckle.AspNetCore`
_Instal Tool Global Entity Framework (EF)_
`dotnet tool install --global dotnet-ef`

## 4. Arsitektur Kode (Models, Data, & DTOs)

### 4.1 Langkah 1: Membuat Entity (Model)

Entity adalah representasi tabel database dalam bentuk kode C#. **File:** _Models/Booking.cs_

```C#
public class Booking {
    public Guid Id { get; set; } = Guid.NewGuid();
    public string NIM { get; set; } = string.Empty;
    public string NamaMahasiswa { get; set; } = string.Empty;
    public string NamaRuangan { get; set; } = string.Empty;
    public DateTime WaktuMulai { get; set; }
    public DateTime WaktuSelesai { get; set; }
    public string Keperluan { get; set; } = string.Empty;
    public string Status { get; set; } = "Pending";
}
```

### 4.2 Langkah 2: Membuat Data Context

Jembatan utama antara kode dan database. **File:** _Data/AppDbContext.cs_

```C#
public class AppDbContext : DbContext {
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Booking> Bookings { get; set; }
}
```

### 4.3 Langkah 3: Membuat DTO (Data Transfer Object)

Agar user tidak perlu menginput ID (karena ID dibuat otomatis oleh sistem). **File:** _DTOs/BookingDto.cs_

```C#
public class BookingDto {
    public string NIM { get; set; } = "";
    public string NamaMahasiswa { get; set; } = "";
    public string NamaRuangan { get; set; } = "";
    public DateTime WaktuMulai { get; set; }
    public DateTime WaktuSelesai { get; set; }
    public string Keperluan { get; set; } = "";
}
```

## 5. Konfigurasi Koneksi & Program.cs

### 5.1 Setting Connection String

Buka _appsettings.json_ dan tambahkan alamat database:

```json
"ConnectionStrings": {
  "DefaultConnection": "Host=localhost; Port=5432; Database=SparkPensDb; Username=postgres; Password=PASSWORD_KAMU"
}
```

### 5.2 Konfigurasi Program.cs

Daftarkan layanan database dan Swagger di _Program.cs:_

```c#
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// ...
app.UseSwagger();
app.UseSwaggerUI();
app.MapControllers();
```

## 6. Migrasi Database (Tugas 6)

Migrasi adalah cara mengubah kode C# menjadi tabel asli di PostgreSQL. **(jalankan di terminal)**

1. Membuat file blueprint migrasi
   `dotnet ef migrations add InitialCreate`
2. Menerapkan ke database asli
   `dotnet ef database update`

## 7. Membuat Controller CRUD (Tugas 7)

Terakhir, kita buat "pintu" agar dunia luar bisa mengakses data kita. **File:** _Controllers/BookingsController.cs_
Tujuan endpoint:

- **GET** _/api/Bookings:_ Mengambil semua data.
- **POST** _/api/Bookings:_ Menambah data baru menggunakan DTO.
- **DELETE** _/api/Bookings/{id}:_ Menghapus data berdasarkan ID.

## 8. Cara Menjalankan & Testing

1. Di terminal, jalankan: `dotnet run`
2. Buka browser: http://localhost:5151/swagger
3. Klik tombol "Try it out" pada endpoint POST untuk mencoba memasukkan data.
