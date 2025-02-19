# 模型创建脚本

Hyperf 提供了创建模型的命令，您可以很方便的根据数据表创建对应模型。命令通过 `AST` 生成模型，所以当您增加了某些方法后，也可以使用脚本方便的重置模型。

```bash
php bin/hyperf.php gen:model table_name
```

## 创建模型

可选参数如下：

|        参数        |  类型  |              默认值               |                       备注                        |
| :----------------: | :----: | :-------------------------------: | :-----------------------------------------------: |
|       --pool       | string |             `default`             |       连接池，脚本会根据当前连接池配置创建        |
|       --path       | string |            `app/Model`            |                     模型路径                      |
|   --force-casts    |  bool  |              `false`              |             是否强制重置 `casts` 参数             |
|      --prefix      | string |             空字符串              |                      表前缀                       |
|   --inheritance    | string |              `Model`              |                       父类                        |
|       --uses       | string | `Hyperf\DbConnection\Model\Model` |              配合 `inheritance` 使用              |
| --refresh-fillable |  bool  |              `false`              |             是否刷新 `fillable` 参数              |
|  --table-mapping   | array  |               `[]`                | 为表名 -> 模型增加映射关系 比如 ['users:Account'] |
|  --ignore-tables   | array  |               `[]`                |        不需要生成模型的表名 比如 ['users']        |
|  --with-comments   |  bool  |              `false`              |                 是否增加字段注释                  |
|  --with-ide        |  bool  |              `false`              |               生成对应的`IDE`文件             |
|  --property-case   |  int   |                `0`                |              字段类型 0 蛇形 1 驼峰               |

当使用 `--property-case` 将字段类型转化为驼峰时，还需要手动在模型中加入 `Hyperf\Database\Model\Concerns\CamelCase`。

对应配置也可以配置到 `databases.{pool}.commands.gen:model` 中，如下

> 中划线都需要转化为下划线

```php
<?php

declare(strict_types=1);

use Hyperf\Database\Commands\ModelOption;

return [
    'default' => [
        // 忽略其他配置
        'commands' => [
            'gen:model' => [
                'path' => 'app/Model',
                'force_casts' => true,
                'inheritance' => 'Model',
                'uses' => '',
                'refresh_fillable' => true,
                'table_mapping' => [],
                'with_comments' => true,
                'property_case' => ModelOption::PROPERTY_SNAKE_CASE,
            ],
        ],
    ],
];
```

创建的模型如下

```php
<?php

declare(strict_types=1);

namespace App\Model;

use Hyperf\DbConnection\Model\Model;

/**
 * @property $id
 * @property $name
 * @property $gender
 * @property $created_at
 * @property $updated_at
 */
class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['id', 'name', 'gender', 'created_at', 'updated_at'];

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = ['id' => 'integer', 'gender' => 'integer'];
}
```

## Visitors

框架提供了几个 `Visitors`，方便用户对脚本能力进行扩展。使用方法很简单，只需要在 `visitors` 配置中，添加对应的 `Visitor` 即可。

```php
<?php

declare(strict_types=1);

return [
    'default' => [
        // 忽略其他配置
        'commands' => [
            'gen:model' => [
                'visitors' => [
                    Hyperf\Database\Commands\Ast\ModelRewriteKeyInfoVisitor::class
                ],
            ],
        ],
    ],
];
```

### 可选 Visitors

- Hyperf\Database\Commands\Ast\ModelRewriteKeyInfoVisitor

此 `Visitor` 可以根据数据库中主键，生成对应的 `$incrementing` `$primaryKey` 和 `$keyType`。

- Hyperf\Database\Commands\Ast\ModelRewriteSoftDeletesVisitor

此 `Visitor` 可以根据 `DELETED_AT` 常量判断该模型是否含有软删除字段，如果存在，则添加对应的 Trait `SoftDeletes`。

- Hyperf\Database\Commands\Ast\ModelRewriteTimestampsVisitor

此 `Visitor` 可以根据 `created_at` 和 `updated_at` 自动判断，是否启用默认记录 `创建和修改时间` 的功能。
