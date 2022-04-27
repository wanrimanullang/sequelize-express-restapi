# Membuat RESTful API menggunakan Express dan Sequelize

## Preparation
Sebelum memulai membuat project, pastikan di laptop / komputer teman-teman sudah terinstall NodeJS dan NPM
Pertama yang harus kita lakukan adalah menginstall secara global Express-generator dan Sequelize-cli. Kalian bisa juga menginstall modul express dan menyimpannya di package.json, namun menggunakan express-generator membuatnya agak lebih mudah.

```
shell
npm i -g express-generator sequelize-cli
```

## Start Project
Pertama, kita harus generate aplikasi yang ingin kita bangun

```
shell
express myapp
```
Nah setelah menjalankan kode di atas, maka akan muncul folder myapp. Lalu kita masuk ke folder tersebut via terminal kemudian jalankan perintah
```
shell
npm install
```

perintah di atas bertujuan untuk menginstall semua module yang ada di package.json.Nah sampai disini kalian sudah bisa menjalankan aplikasi yang sudah dibuat. Caranya dengan menuliskan perintah

```
shell
npm start
```
<br>kemudian buka browser kalian dan akses localhost:3000
<img src="/public/images/express.png"><br>

Kalau muncul tampilan seperti itu, maka aplikasi kalian sudah bisa diakses. Untuk menghentikan aplikasi yang sedang berjalan, maka masuk ke terminal dan masukkan kombinasi Ctrl+C

## Koneksi ke DB
disini saya membuat database dengan postgres. kita akan coba untuk mengkoneksikan nya dengan database

```shell
npm install sequelize mysql2 --save
```
Perintah save di atas bertujuan untuk menyimpan modul yang kita pakai tadi ke file package.json
Setelah itu, kita akan membuat direktori yang diperlukan untuk koneksi ke DB dengan perintah

```shell
sequelize init
```
Perintah di atas menghasilkan 4 folder, yaitu config, models, migrations, dan seeders.
Untuk mengkonfigurasikan ke DB, bukalah file config/config.json. Disitu tertera 3 config ke DB, karena kita sedang di tahap development maka kita akan mengkonfigurasikan koneksi DB di config development.

```
javascript
  "development": {
    "username": " ", //username database
    "password": " ", //password database
    "database": " ", //nama database
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
```

## Membuat Models, dan Migrations
Sebelum memulai membuat API Endpoint, kita harus membuat model sebagai penghubung aplikasi ke DB, buka terminal dan tuliskan perintah berikut

```
shell
sequelize model:create --name users --attributes name:string,email:string,phone_number:string,gender:boolean
```

Perintah di atas akan men-generate file model di folder models dan file migration di folder migrations. Bukalah file migrations dan edit sesuai dengan kebutuhan kita

```
javascript

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER(5)
      },
      name: {
        type: Sequelize.STRING(30)
      },
      email: {
        type: Sequelize.STRING(50)
      },
      phone_number: {
        type: Sequelize.STRING(15)
      },
      gender: {
        type: Sequelize.BOOLEAN,
        defaultValue: true
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: Sequelize.fn('NOW'),
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE,
        defaultValue: Sequelize.fn('NOW'),
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('users');
  }
};
```
Kita bisa memberikan length dari setiap field dan default valuenya. Untuk menjalankan file migrations di atas, jalan perintah di terminal

```shell
sequelize db:migrate
```
Setelah itu cek DB yang sudah kita buat, maka table users sudah terbuat.

# Pembuatan Routes API
Pertama kita buat terlebih dahulu file users.js di folder routes

```
javascript
const express = require('express');
const router = express.Router();
const model = require('../models/index');
// GET users listing.
router.get('/', function(req, res, next) {
});
// POST users
router.post('/', function(req, res, next) {
});
// UPDATE users
router.patch('/:id', function(req, res, next) {
});
// DELETE users
router.delete('/:id', function(req, res, next) {
});
module.exports = router;
Lalu di file app.js kita konfigurasi agar app.js bisa memanggil route users
.................
.....................
.................
const users = require('./routes/users');
.................
.....................
.................
app.use('/users', users);
................
......................
................
Get All Users
Tambahkan baris kode berikut pada file routes/users.js
router.get('/', async function (req, res, next) {
  try {
    const users = await model.users.findAll({});
    if (users.length !== 0) {
      res.json({
        'status': 'OK',
        'messages': '',
        'data': users
      })
    } else {
      res.json({
        'status': 'ERROR',
        'messages': 'EMPTY',
        'data': {}
      })
    }
  } catch (err) {
    res.json({
      'status': 'ERROR',
      'messages': err.message,
      'data': {}
    })
  }
});
```

Setelah selesai, coba jalankan perintah npm start, kemudian menggunakan aplikasi Postman akses menggunakan method GET dengan url localhost:3000/users
## Create User
Tambahkan baris kode berikut pada file routes/users.js

```
javascript
router.post('/', async function (req, res, next) {
  try {
    const {
      name,
      email,
      gender,
      phoneNumber
    } = req.body;
    const users = await model.users.create({
      name,
      email,
      gender,
      phone_number: phoneNumber
    });
  if (users) {
    res.status(201).json({
      'status': 'OK',
      'messages': 'User berhasil ditambahkan',
      'data': users,
    })
  }
 } catch (err) {
   res.status(400).json({
     'status': 'ERROR',
     'messages': err.message,
     'data': {},
   })
 }
});
```

Setelah selesai, coba jalankan perintah npm start, jika aplikasi sudah berjalan maka restart aplikasi kemudian menggunakan aplikasi Postman akses menggunakan method POST dengan url localhost:3000/users. Pada body pilih raw dan JSON(application/json).

## Update User
Tambahkan baris kode berikut pada file routes/users.js

```
javascript

router.patch('/:id', async function (req, res, next) {
  try {
    const usersId = req.params.id;
    const {
      name,
      email,
      gender,
      phoneNumber
    } = req.body;
    const users = await model.users.update({
      name,
      email,
      gender,
      phone_number: phoneNumber
    }, {
      where: {
        id: usersId
      }
    });
    if (users) {
      res.json({
        'status': 'OK',
        'messages': 'User berhasil diupdate',
        'data': users,
      })
    }
  } catch (err) {
    res.status(400).json({
      'status': 'ERROR',
      'messages': err.message,
      'data': {},
    })
  }
});
```

Setelah selesai, coba jalankan perintah npm start, jika aplikasi sudah berjalan maka restart aplikasi kemudian menggunakan aplikasi Postman akses menggunakan method PATCH dengan url localhost:3000/users/{id_user}. Pada body pilih raw dan JSON(application/json).

# Delete User
Tambahkan baris kode berikut pada file routes/users.js

```
javascript

router.delete('/:id', async function (req, res, next) {
  try {
    const usersId = req.params.id;
    const users = await model.users.destroy({ where: {
      id: usersId
    }})
    if (users) {
      res.json({
        'status': 'OK',
        'messages': 'User berhasil dihapus',
        'data': users,
      })
    }
  } catch (err) {
    res.status(400).json({
      'status': 'ERROR',
      'messages': err.message,
      'data': {},
    })
  }
});
```
Setelah selesai, coba jalankan perintah npm start, jika aplikasi sudah berjalan maka restart aplikasi kemudian menggunakan aplikasi Postman akses menggunakan method DELETE dengan url localhost:3000/users/{id_user}. Pada body pilih raw dan JSON(application/json).



