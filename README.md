# gRPC Polyglot Lab

Praktikum komunikasi antar layanan (*inter-service*) dengan **gRPC**: satu **server Go** dipanggil oleh dua **client** berbeda bahasa, yaitu **Python** dan **Node.js**. Ketiganya memakai satu kontrak yang sama, yaitu `user.proto`.

## Cara kerja singkat

```
Client Python  ──┐
                 ├──►  Server Go (port 50051)  ──►  mengembalikan profil user
Client Node.js ──┘
```

Client mengirim `user_id`, server membalas `name`, `email`, dan `phone`.
Data profil hanya ditulis satu kali di server Go, tetapi tampil di kedua client.

## Struktur project

```
grpc-polyglot-lab/
├── user.proto          # kontrak API (dipakai bersama)
├── README.md
├── go-server/          # server gRPC (Go)
│   ├── main.go
│   └── pb/             # kode Go hasil generate dari user.proto
├── python-client/      # client gRPC (Python)
│   ├── client.py
│   └── user_pb2*.py    # kode Python hasil generate
└── node-client/        # client gRPC (Node.js)
    ├── client.js
    └── package.json
```

## Prasyarat

Pastikan perangkat lunak berikut sudah terpasang. Cek dengan perintahnya:

| Perangkat lunak | Perintah cek |
|---|---|
| Git | `git --version` |
| Go | `go version` |
| Python 3 | `python --version` (atau `python3 --version`) |
| Node.js dan npm | `node -v` dan `npm -v` |

`protoc` **tidak wajib** untuk sekadar menjalankan project ini, karena kode hasil generate sudah ada di repo. `protoc` baru diperlukan jika Anda mengubah `user.proto` (lihat bagian *Mengubah kontrak*).

## Langkah menjalankan

### 1. Ambil kode

```bash
git clone https://github.com/muftikhalis95/grpc-polyglot-lab.git
cd grpc-polyglot-lab
```

### 2. Jalankan server Go (Terminal 1)

```bash
cd go-server
go mod tidy
go run main.go
```

Jika berhasil, muncul:

```
gRPC Server (Go) berjalan di port :50051...
```

**Biarkan terminal ini tetap terbuka.** Server harus aktif saat client dijalankan.

### 3. Jalankan client Python (Terminal 2)

Buka terminal baru, masuk ke folder proyek, lalu:

```bash
cd python-client
python -m venv .venv
source .venv/bin/activate        # Windows (Git Bash): source .venv/Scripts/activate
pip install grpcio grpcio-tools
python client.py
```

Hasil yang benar:

```
Mengirim request ke Go Server menggunakan Python...
Respon dari Server -> Nama: Budi Santoso, Email: budi@universitas.ac.id, Telepon: +62-812-3456-7890
```

### 4. Jalankan client Node.js (Terminal 3)

Buka terminal baru, masuk ke folder proyek, lalu:

```bash
cd node-client
npm install
node client.js
```

Hasil yang benar:

```
Mengirim request ke Go Server menggunakan Node.js...
Respon dari Server -> Nama: Budi Santoso, Email: budi@universitas.ac.id, Telepon: +62-812-3456-7890
```

### 5. Cek log server

Pada Terminal 1 akan muncul satu baris log untuk setiap client yang memanggil:

```
Menerima request untuk User ID: 1
```

## Mengubah kontrak (`user.proto`)

Jika Anda mengubah `user.proto` (misalnya menambah field), kode Go dan Python harus dibuat ulang. Node.js tidak perlu karena membaca `user.proto` langsung saat berjalan.

Pasang `protoc` (Linux Debian/Ubuntu):

```bash
sudo apt update && sudo apt install -y protobuf-compiler
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
export PATH=$PATH:$(go env GOPATH)/bin
```

Generate ulang (jalankan dari folder utama proyek):

```bash
# Go
cd go-server
protoc --proto_path=../ --go_out=. --go-grpc_out=. ../user.proto
cd ..

# Python
cd python-client
source .venv/bin/activate
python -m grpc_tools.protoc --proto_path=../ --python_out=. --grpc_python_out=. ../user.proto
cd ..
```

Setelah itu **restart server Go** agar memakai kode yang baru.

## Masalah yang sering muncul

| Masalah | Penyebab dan solusi |
|---|---|
| `Connection refused` atau `UNAVAILABLE` | Server Go belum berjalan. Jalankan Langkah 2 lebih dulu. |
| `ModuleNotFoundError: user_pb2` | Jalankan `client.py` dari dalam folder `python-client`. |
| `Cannot find module '@grpc/grpc-js'` | Jalankan `npm install` di folder `node-client`. |
| `protoc-gen-go: program not found` | Tambahkan folder `~/go/bin` ke PATH (lihat bagian *Mengubah kontrak*). |
| `Telepon:` kosong | Server Go belum di-restart setelah kode diubah. |
| Port 50051 sudah dipakai | Hentikan program lain yang memakai port tersebut. |

## Analisis: Static Code Generation vs Dynamic Runtime Generation

Pada pendekatan *static code generation* (Go dan Python), file `.proto` diproses sebelum program dijalankan menggunakan `protoc`, yang menghasilkan file kode nyata (`user.pb.go`, `user_pb2.py`) yang kemudian di-import langsung. Keuntungannya, kesalahan tipe dapat terdeteksi lebih awal dan ada autocomplete di editor, tetapi setiap perubahan `.proto` mewajibkan generate ulang. Pada *dynamic runtime generation* (Node.js), library `proto-loader` membaca `user.proto` saat program berjalan dan membangun definisinya di memori, sehingga tidak ada langkah build dan perubahan langsung berlaku setelah program dijalankan ulang. Kekurangannya, tidak ada pengecekan tipe saat kompilasi dan file `.proto` harus tersedia saat runtime.

| Aspek | Static (Go dan Python) | Dynamic (Node.js) |
|---|---|---|
| Waktu pemrosesan `.proto` | Sebelum program dijalankan, lewat `protoc` | Saat program berjalan, lewat `proto-loader` |
| File hasil generate | Ada | Tidak ada |
| Saat `.proto` berubah | Wajib generate ulang | Cukup jalankan ulang program |
| Pengecekan tipe | Lebih awal (saat kompilasi di Go) | Saat program berjalan |

## Penulis

Mufti Khalis, TRKJ 2C, Politeknik Negeri Lhokseumawe.
