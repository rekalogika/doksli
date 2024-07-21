---
title: Repository
---

Repository pattern adalah pola design yang memberikan abstraksi untuk mengakses
data. Kita menggunakan repository untuk mengambil dan menyimpan data ke dalam
database.

Hal tersebut dilakukan melalui satu pintu. Keluar masuk entity dari dan ke
database dilakukan hanya melalui repository. Apabila ada perubahan cara
penyimpanan data, hal tersebut mudah dilakukan karena hanya perlu mengubah
implementasi repository.

## Membuat Interface Repository

Pada arsitektur yang kita gunakan, repository berada di layer domain service.
Karena repository berhubungan dengan infrastruktur, kita hanya akan membuat
interfacenya saja di layer domain service:

```php title="src/DomainService/Repository/PostRepository.php"
namespace App\DomainService\Repository;

use App\Domain\Entity\Post;
use Rekalogika\Contracts\Collections\Repository;

/**
 * @extends Repository<string,Post>
 */
interface PostRepository extends Repository
{
}
```

:::info

Kita selalu menggunakan string UUID untuk primary key. Dengan demikian, template
`TKey` dalam interface repository selalu berupa `string`.

:::

Sedangkan untuk `Comment`:

```php title="src/DomainService/Repository/CommentRepository.php"
namespace App\DomainService\Repository;

use App\Domain\Entity\Comment;
use Rekalogika\Contracts\Collections\Repository;

/**
 * @extends Repository<string,Comment>
 */
interface CommentRepository extends Repository
{
}
```

## Membuat Implementasi Repository

Setelah itu kita buatkan implementasinya.

Untuk `Post`:

```php title="src/Infrastructure/Repository/PostRepository.php"
namespace App\Infrastructure\Repository;

use App\Domain\Entity\Post;
use App\DomainService\Repository\PostRepository;
use Doctrine\Common\Collections\Order;
use Doctrine\Persistence\ManagerRegistry;
use Rekalogika\Collections\ORM\AbstractRepository;

/**
 * @extends AbstractRepository<string,Post>
 */
class PostRepository extends AbstractRepository implements PostRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct(
            managerRegistry: $registry,
            class: Post::class,
            orderBy: ['id' => Order::Descending],
        );
    }
}
```

Untuk `Comment`:

```php title="src/Infrastructure/Repository/CommentRepository.php"
namespace App\Infrastructure\Repository;

use App\Domain\Entity\Comment;
use App\DomainService\Repository\CommentRepository;
use Doctrine\Common\Collections\Order;
use Doctrine\Persistence\ManagerRegistry;
use Rekalogika\Collections\ORM\AbstractRepository;

/**
 * @extends AbstractRepository<string,Comment>
 */
class CommentRepository extends AbstractRepository implements CommentRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct(
            managerRegistry: $registry,
            class: Comment::class,
            orderBy: ['id' => Order::Descending],
        );
    }
}
```
