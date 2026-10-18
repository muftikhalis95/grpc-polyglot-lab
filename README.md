# gRPC Polyglot Lab

Server gRPC (Go) dengan client Python dan Node.js, memakai satu kontrak `user.proto`.

## Analisis: Static Code Generation vs Dynamic Runtime Generation

Pada pendekatan static code generation (Go dan Python), file `.proto` diproses sebelum program dijalankan menggunakan `protoc`, yang menghasilkan file kode nyata (`user.pb.go`, `user_pb2.py`) yang kemudian di-import langsung. Keuntungannya, kesalahan tipe dapat terdeteksi lebih awal dan ada autocomplete di editor, tetapi setiap perubahan `.proto` mewajibkan generate ulang. Pada dynamic runtime generation (Node.js), library `proto-loader` membaca `user.proto` saat program berjalan dan membangun definisinya di memori, sehingga tidak ada langkah build dan perubahan langsung berlaku setelah program dijalankan ulang. Kekurangannya, tidak ada pengecekan tipe saat kompilasi dan file `.proto` harus tersedia saat runtime.
