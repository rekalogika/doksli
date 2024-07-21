---
title: Domain Testing
---

Setelah ada domain yang sudah diimplementasikan, kita buat unit test sederhana.
Biasanya bisa dengan cepat dibuat dengan AI. Contoh:

```php title="tests/Domain/Entity/PostTest.php"
namespace App\Tests\Domain\Entity;

use App\Domain\Post;
use PHPUnit\Framework\TestCase;

class PostTest extends TestCase
{
    public function testCreatePost(): void
    {
        $post = new Post('Title', 'Content');

        $this->assertEquals('Title', $post->getTitle());
        $this->assertEquals('Content', $post->getContent());
    }

    public function testUpdatePost(): void
    {
        $post = new Post('Title', 'Content');
        $post->setTitle('New Title');
        $post->setContent('New Content');

        $this->assertEquals('New Title', $post->getTitle());
        $this->assertEquals('New Content', $post->getContent());
    }

    // tbd
}
```
