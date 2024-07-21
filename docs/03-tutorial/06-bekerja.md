---
title: Bekerja Dengan Entity
---

Setelah membuat domain, mapping database, dan repositorynya, kita sudah bisa
bekerja dengan entity tersebut.

## Membuat `Post` Baru dan Menyimpannya ke Database

```php
use App\Domain\Entity\Post;
use App\DomainService\Repository\PostRepository;
use Doctrine\ORM\EntityManagerInterface;

/** @var PostRepository $postRepository */
/** @var EntityManagerInterface $entityManager */

$post = new Post('Title', 'Content');
$postRepository->add($post);
$entityManager->flush();
```

## Mengambil `Post` dari Database

```php
use App\Domain\Entity\Post;
use App\DomainService\Repository\PostRepository;

/** @var PostRepository $postRepository */

// $post akan null jika tidak ditemukan di database
$post = $postRepository->get('91b2679e-47a5-11ef-b06f-8c8caab77b0f');

// akan throw exception jika tidak ditemukan di database, dan otomatis
// menjadi error 404 di browser
$post = $postRepository->fetch('91b2679e-47a5-11ef-b06f-8c8caab77b0f');
```

## Mengubah `Post` dan Menyimpannya ke Database

```php
use App\Domain\Entity\Post;
use App\DomainService\Repository\PostRepository;

/** @var PostRepository $postRepository */
/** @var EntityManagerInterface $entityManager */

$post = $postRepository->get('91b2679e-47a5-11ef-b06f-8c8caab77b0f');
$post->setTitle('New Title');
$post->setContent('New Content');
$entityManager->flush();
```

## Menghapus `Post` dari Database

```php
use App\Domain\Entity\Post;
use App\DomainService\Repository\PostRepository;
use Doctrine\ORM\EntityManagerInterface;

/** @var PostRepository $postRepository */
/** @var EntityManagerInterface $entityManager */

// cara pertama
$post = $postRepository->get('91b2679e-47a5-11ef-b06f-8c8caab77b0f');
$postRepository->removeElement($post);
$entityManager->flush();

// cara kedua
$postRepository->remove('91b2679e-47a5-11ef-b06f-8c8caa77b0f');
$entityManager->flush();
```

## Melakukan Iterasi Terhadap Semua `Post`

```php
use App\DomainService\Repository\PostRepository;
use Doctrine\ORM\EntityManagerInterface;

/** @var PostRepository $postRepository */
/** @var EntityManagerInterface $entityManager */

// cara langsung, akan mengambil semua data dari database sekaligus, dan
// berpotensi out of memory jika data terlalu banyak:

foreach ($postRepository as $post) {
    // lakukan sesuatu dengan $post
}

// menggunakan batch, akan mengambil data per halaman, dan tidak akan
// mengalami out of memory:

foreach ($postRepository->withItemsPerPage(1000)->getPages() as $page) {
    foreach ($page as $post) {
        // lakukan sesuatu dengan $post
    }

    $entityManager->clear();
}
```