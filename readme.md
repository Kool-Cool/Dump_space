# this will PDSA dump space


# Issue - 1 
(while migrating tables)
```shell
 2024_06_04_171313_create_user_module_notifications_table .......................................................... 23.57ms FAIL

   Illuminate\Database\QueryException 

  SQLSTATE[42000]: Syntax error or access violation: 1072 Key column 'channel_id' doesn't exist in table (Connection: mysql, SQL: create table `user_module_notifications` (`user_id` bigint unsigned not null, `module_id` bigint unsigned not null, `notification_channel_id` bigint unsnel_id` bigint unsigned not null, `active` tinyint(1) not null default '1', `created_at` timestamp null, `updated_at` timestamp null, primary key (`user_id`, `module_id`, `channel_id`)) default character set utf8mb4 collate 'utf8mb4_unicode_ci')

  at vendor\laravel\framework\src\Illuminate\Database\Connection.php:813
    809▕                     $this->getName(), $query, $this->prepareBindings($bindings), $e
    810▕                 );
    811▕             }
    812▕
  ➜ 813▕             throw new QueryException(
    814▕                 $this->getName(), $query, $this->prepareBindings($bindings), $e
    815▕             );
    816▕         }
    817▕     }

  1   vendor\laravel\framework\src\Illuminate\Database\Connection.php:571
      PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1072 Key column 'channel_id' doesn't exist in table")

  2   vendor\laravel\framework\src\Illuminate\Database\Connection.php:571
      PDOStatement::execute()
```

 **FIX** 
```php

<?php
// database\migrations\2024_06_04_171313_create_user_module_notifications_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUserModuleNotificationsTable extends Migration
{
    public function up(): void
    {
        Schema::create('user_module_notifications', function (Blueprint $table) {
            $table->unsignedBigInteger('user_id');
            $table->unsignedBigInteger('module_id');
            $table->unsignedBigInteger('notification_channel_id'); // Corrected column name
            $table->tinyInteger('active')->default(1);
            $table->timestamps();

            // Add foreign key constraints
            $table->foreign('user_id')->references('id')->on('users');
            $table->foreign('module_id')->references('id')->on('modules');
            $table->foreign('notification_channel_id')->references('id')->on('notification_channels');
            
            // Primary key definition
            $table->primary(['user_id', 'module_id', 'notification_channel_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('user_module_notifications');
    }
}


```

# Issue - 2
(while seeding the tables)

```php render
  Database\Seeders\UserSeeder ..................................................................................................... RUNNING  

   Illuminate\Database\QueryException 

  SQLSTATE[42000]: Syntax error or access violation: 1701 Cannot truncate a table referenced in a foreign key constraint (`dev_credit_compass`.`user_module_notifications`, CONSTRAINT `user_module_notifications_user_id_foreign` FOREIGN KEY (`user_id`) REFERENCES `dev_credit_compass`.`users` (`id`)) (Connection: mysql, SQL: truncate table `users`)

  at vendor\laravel\framework\src\Illuminate\Database\Connection.php:813
    809▕                     $this->getName(), $query, $this->prepareBindings($bindings), $e
    810▕                 );
    811▕             }
    812▕
  ➜ 813▕             throw new QueryException(
    814▕                 $this->getName(), $query, $this->prepareBindings($bindings), $e
    815▕             );
    816▕         }
    817▕     }

  1   vendor\laravel\framework\src\Illuminate\Database\Connection.php:571
      PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1701 Cannot truncate a table referenced in a foreign key constraint (`dev_credit_compass`.`user_module_notifications`, CONSTRAINT `user_module_notifications_user_id_foreign` FOREIGN KEY (`user_id`) REFERENCES `dev_credit_compass`.`users` (`id`))")

  2   vendor\laravel\framework\src\Illuminate\Database\Connection.php:571
      PDOStatement::execute()

```
 **FIX** 
```php
<?php
// database\seeders\UserSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        // Disable foreign key checks
        DB::statement('SET FOREIGN_KEY_CHECKS=0;');


        DB::table('users')->truncate();

        $users = [
            [
                'user_type_id' => 1,
                'firstname' => 'John',
                'lastname'  => 'Doe',
                'email'     => 'john@gmail.com',
                'mobile_no' => '9999999999',
                'city'      => 'Mumbai',
                'pincode'   => '400059',
                'address'   => 'Mumbai, Mumbai',
                'password'  => Hash::make('password'),
            ],
        ];

        DB::table('users')->insert($users);

        // Re-enable foreign key checks
        DB::statement('SET FOREIGN_KEY_CHECKS=1;');
    
    }
}

```




