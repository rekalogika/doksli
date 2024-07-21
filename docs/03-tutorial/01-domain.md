---
title: Domain
---

Untuk memulai, pertama-tama kita perlu desain domain-nya. Komponen utama domain
adalah entity dan value object. Bedanya, entity memiliki state, dan biasanya
akan disimpan di database. Sedangkan value object tidak memiliki state.

:::caution

Pada tahap ini kita tidak perlu memikirkan teknologi yang digunakan, seperti
database dan sebagainya. Kita hanya perlu fokus pada model bisnisnya saja.

:::

## Entity `Post`

Berikut contoh entity untuk `Post`:

```php
namespace App\Domain;

use Rekalogika\CommonBundle\Domain\Entity\AbstractAggregateRoot;
use Symfony\Component\Clock\DatePoint;

class Post extends AbstractAggregateRoot
{
    private \DateTimeInterface $createdTime;
    private string $title;
    private string $content;

    public function __construct(
        string $title,
        string $content,
    ) {
        $this->title = $title;
        $this->createdTime = new DatePoint();
    }
}
```

Terlihat bahwa class ini sudah bisa mewakili fungsi dasar dari blog post. Setiap
blog post memiliki judul, konten, dan waktu pembuatan. Waktu pembuatan otomatis
dibuat saat objek dibuat. Sedangkan terlihat `title` dan `content` adalah
property yang wajib ada (tidak boleh kosong), dan harus ada saat objek pertama
kali dibuat.

:::info

`DateTimeInterface` adalah interface dari semua objek waktu di PHP.
Implementasi di PHP berupa `DateTimeImmutable` dan `DateTime`. Di sini kita
menggunakan `DatePoint` dari Symfony Clock. Alasannya karena `DatePoint` lebih
mudah diuji daripada `DateTimeImmutable` dan `DateTime`.

:::

Properti `createdTime` tidak boleh diubah setelah, hanya bisa dilihat, maka kita
jadikan `readonly` dan hanya kita buatkan getternya, tidak perlu setter:

```php
class Post extends AbstractAggregateRoot
{
    private readonly \DateTimeInterface $createdTime;

    public function getCreatedTime(): \DateTimeInterface
    {
        return $this->createdTime;
    }
}
```

:::info

Objek `DateTimeInterface` adalah salah satu contoh dari value object.

:::

Karena `title` dan `content` boleh diubah setelah pos dibuat, maka kita buatkan
getter dan setternya:

```php
class Post extends AbstractAggregateRoot
{
    public function getTitle(): string
    {
        return $this->title;
    }

    public function setTitle(string $title): void
    {
        $this->title = $title;
    }

    public function getContent(): string
    {
        return $this->content;
    }

    public function setContent(string $content): void
    {
        $this->content = $content;
    }
}
```

## Entity `Comment`

Karena setiap pos bisa memiliki komentar, maka kita buat entity komentar:

```php
namespace App\Domain;

use Rekalogika\CommonBundle\Domain\Entity\AbstractEntity;
use Symfony\Component\Clock\DatePoint;

class Comment extends AbstractEntity
{
    private string $content;
    private readonly \DateTimeInterface $createdTime;

    public function __construct(string $content)
    {
        $this->content = $content;
        $this->createdTime = new DatePoint();
    }

    public function getCreatedTime(): \DateTimeInterface
    {
        return $this->createdTime;
    }
}
```

Kurang lebih sama perilakunya dengan `Post`. Komentar memiliki konten dan waktu.

## Relasi Pada Sisi `Post`

Karena setiap pos bisa memiliki beberapa komentar, maka kita perlu membuat
relasinya. Kita bisa membuat property `comments` di `Post`:

```php
use Doctrine\Common\Collections\Collection;
use Rekalogika\Contracts\Collections\Recollection;
use Rekalogika\Domain\Collections\ArrayCollection;
use Rekalogika\Domain\Collections\RecollectionDecorator;

class Post extends AbstractAggregateRoot
{
    /**
     * @var Collection<string,Comment>
     */
    private Collection $comments;

    public function __construct(
        // ...
    ) {
        // ...
        $this->comments = new ArrayCollection();
    }

    public function addComment(Comment $comment): void
    {
        $this->getComments()->add($comment);
        $comment->setPost($this);
    }

    /**
     * @return Recollection<string,Comment>
     */
    public function getComments(): Recollection
    {
        return RecollectionDecorator::create(
            collection: $this->insurables,
            count: new PrecountingStrategy($this->insurablesCount),
            indexBy: 'id',
            orderBy: ['id' => Order::Descending]
        );
    }

    public function removeComment(Comment $comment): void
    {
        if ($this->getComments()->removeElement($comment)) {
            if ($comment->getPost() === $this) {
                $comment->setPost(null);
            }
        }
    }
}
```

Yang perlu diperhatikan:

* Untuk menampung komentar, harus menggunakan tipe `Collection`, tidak boleh
  array biasa.
* Di atasnya kita tambahkan `@var Collection<string,Comment>` untuk memberi tahu
  bahwa `comments` adalah collection dari beberapa objek `Comment` dengan key
  bertipe `string`.

:::info Perbedaan Dengan Symfony dan Doctrine Standard

Kita menggunakan `ArrayCollection` versi kita, bukan yang dari Doctrine,
[penjelasannya di
sini](https://rekalogika.dev/collections/implementations/array-collection).

Untuk `getComments()`, kita menggunakan `RecollectionDecorator`, bukan plain
`Collection`. Alasannya:

* Mengimplementasikan `PageableInterface` yang bisa digunakan langsung untuk
  batch processing dan pagination
* Otomatis melakukan limit, sehingga tidak akan menghabiskan memori.

Cek infonya [di sini](https://rekalogika.dev/collections).

Yang mengakses `$this->comments` hanyalah method `getComments()`. Method lain
harus menggunakan method `getComments()` untuk mengakses data komentar. Ini kita
lakukan untuk menghindari kesalahan pemrograman yang dapat mengakibatkan
out-of-memory.

:::

## Relasi Pada Sisi `Comment`

Lalu untuk `Comment`, kita bisa menambahkan property `post`:

```php
class Comment extends AbstractEntity
{
    private ?Post $post = null;

    public function setPost(?Post $post): void
    {
        $this->post = $post;
    }

    public function getPost(): Post
    {
        return $this->post;
    }
}
```

Jika `$post` null, maka komentar tersebut tidak terhubung dengan pos manapun.