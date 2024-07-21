---
title: Konfigurasi ORM
---

Setelah pembuatan domain selesai, baru kita memikirkan bagaimana cara menyimpan
data ke database. Untuk melakukan itu, kita membuat konfigurasi ORM Doctrine
dalam bentuk XML.

## Konfigurasi Untuk `Post`

```xml title="config/doctrine/Post.orm.xml"
<?xml version="1.0" encoding="UTF-8"?>

<doctrine-mapping
        xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
        xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
        xmlns:gedmo="http://gediminasm.org/schemas/orm/doctrine-extensions-mapping"
        xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping https://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

    <entity name="App\Domain\Entity\Post">

        <field
                name="title"
                type="string"
                nullable="false" />

        <field
                name="content"
                type="text"
                nullable="false" />

        <field
                name="createdTime"
                type="datetime"
                nullable="false" />

        <one-to-many
                field="comments"
                target-entity="App\Domain\Entity\Comment"
                mapped-by="post"
                index-by="id"
                fetch="EXTRA_LAZY">
        </one-to-many>

    </entity>
</doctrine-mapping>
```

:::caution Perbedaan Dengan Doctrine Standar

Umumnya, konfigurasi Doctrine akan mendefinisikan class repository. Namun kita
tidak mendefinisikan repository di sini, karena kita akan menggunakan
implementasi repositori buatan kita sendiri.

:::

Di sini semua field-field adalah nama property dari class `Post`, beserta
definisinya di database.

Doctrine akan secara otomatis memberikan nama tabel dan nama kolom yang sesuai
dengan konvensi yang digunakan. Untuk sistem kita, Doctrine akan menggunakan
snake case dengan prefix `t_` untuk nama tabel, sehingga untuk entity `Post`
akan disimpan ke tabel `t_post`.

Sedangkan untuk nama kolom menggunakan aturan sama, tetapi dengan prefix `c_`.
Sehingga field `title` akan disimpan ke kolom `c_title`.

## Konfigurasi Untuk `Comment`

```xml title="config/doctrine/Comment.orm.xml"
<?xml version="1.0" encoding="UTF-8"?>

<doctrine-mapping
        xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
        xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
        xmlns:gedmo="http://gediminasm.org/schemas/orm/doctrine-extensions-mapping"
        xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping https://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

    <entity name="App\Domain\Entity\Comment">

        <field
                name="content"
                type="text"
                nullable="false" />

        <field
                name="createdTime"
                type="datetime"
                nullable="false" />

        <many-to-one
                field="post"
                inversed-by="comments"
                fetch="LAZY">
        </many-to-one>

    </entity>