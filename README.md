# ci4_codeigniter4_crud_simple
 
https://makitweb.com/crud-create-read-update-delete-in-a-codeigniter-4/

https://onlinewebtutorblog.com/codeigniter-4-command-change-php-spark-default-port/


# Content

- [Database configuration]()
- [Create Table]()
- [Model]()
- [Route]()
- [Controller]()
- [View]()
- [Run]()
- [Output]()

---

## Database configuration

- Open `.env` file which is available at the project root.

- Remove # from start of database.default.hostname, database.default.database, database.default.username, database
default.password, and database.default.DBDriver.
- Update the configuration and save it.

```
database.default.hostname = 127.0.0.1
database.default.database = testdb
database.default.username = root
database.default.password = 
database.default.DBDriver = MySQLi
```

#
## Create Table

- Create a new table `subjects` using migration.

```
php spark migrate:create create_subjects_table
```

- Now, navigate to `app/Database/Migrations/` folder from the project root.
- Find a PHP file that ends with `create_subjects_table` and open it.
- Define the table structure in the `up()` method.
- Using the `down()` method delete `subjects` table which calls when undoing migration.

```php
<?php namespace App\Database\Migrations;

use CodeIgniter\Database\Migration;

class CreateSubjectsTable extends Migration
{
    public function up() {
       $this->forge->addField([
         'id' => [
            'type' => 'INT',
            'constraint' => 5,
            'unsigned' => true,
            'auto_increment' => true,
         ],
         'name' => [
            'type' => 'VARCHAR',
            'constraint' => '100',
         ],
         'description' => [
            'type' => 'TEXT',
            'null' => true,
         ],
       ]);
       $this->forge->addKey('id', true);
       $this->forge->createTable('subjects');
    }

    //--------------------------------------------------------------------

    public function down() {
       $this->forge->dropTable('subjects');
    }
}
```

- Run the migration.

```
php spark migrate
```

#
## Model

- Create `Subjects.php` file in `app/Models/ folder`.
- Open the file.
- Specify table name `"subjects"` in `$table` variable, primary key `"id"` in` $primaryKey`, Return type `"array"` in `$returnType`.
- In `$allowedFields` Array specify field names – `['name', 'description']` that can be set during insert and update.

```php
<?php 
namespace App\Models;

use CodeIgniter\Model;

class Subjects extends Model
{
    protected $table = 'subjects'; 
    protected $primaryKey = 'id';

    protected $returnType = 'array';

    protected $allowedFields = ['name', 'description'];
    protected $useTimestamps = false;

    protected $validationRules = [];
    protected $validationMessages = [];
    protected $skipValidation = false;
}
```

#
## Route

- Open app/Config/Routes.php file.
- Define 6 routes –
  - **/ –** Display subject list.
  - **subjects/create –** Open add subject view.
  - **subjects/store –** Submit subject form to insert a record.
  - **subjects/edit/(:num) –** Open edit subject view by id.
  - **subjects/update/(:num) –** Submit edit form to update a record by id.
  - **subjects/delete/(:num) –** Delete a subject by id.

```php
$routes->get('/', 'SubjectsController::index');

$routes->get('subjects/create', 'SubjectsController::create');
$routes->post('subjects/store', 'SubjectsController::store');

$routes->get('subjects/edit/(:num)', 'SubjectsController::edit/$1');
$routes->post('subjects/update/(:num)', 'SubjectsController::update/$1');

$routes->get('subjects/delete/(:num)', 'SubjectsController::delete/$1');
```

#
## Controller

- Create `SubjectsController.php` file in `app/Controllers/` folder.
- Open the file.
- Import `Subjects` Model.
- Create 6 methods –
  - **index() –** Select all records from the subjects table and assign in $data['subjects']. Load subjects/index view and pass $data.
  - **create() –** With this method load `subjects/create` view for adding a new subject.
  - **store() –** With this method insert a new record in the subjects table.

Read POST values. If `'submit'` is POST then define validation. If values are not validated then return to the `subjects/create` view with the error response.

If values are validated then insert a record in the `subjects` table. If a record is successfully inserted then store the success message in `session()->setFlashdata('message')` and class name in session()->setFlashdata('alert-class') and return to the `subjects/create route`.

Similarly, if a record is not inserted then store the failed message in `session()->setFlashdata('message')` and class name in `session()->setFlashdata('alert-class')` and return to the `subjects/create` route.

  - edit() – With this method load edit subject view. Select a record from the `subjects` table by `$id` and assign in `$data['subject']`. Load `subjects/edit` view and pass `$data`.
  - update() – With this method update a record in the `subjects` table.

Read POST values. If `'submit'` is POST then define validation. If values are not validated then return to the edit view with the error response.

If values are not validated then redirect to the edit view with the error response. Select a record from the `subjects` table by `$id` and assign in `$subject`.

If values are validated then update a record by `$id`. If a record is successfully inserted then store the success message in `session()->setFlashdata('message')` and class name in `session()->setFlashdata('alert-class')` and return to the `/` route.

Similarly, if a record is not inserted then store the failed message in `session()->setFlashdata('message')` and class name in `session()->setFlashdata('alert-class')` and return to the `subjects/edit` route.

  - **delete() –** With this method delete a record from the `subjects` table by `$id`.

Check $id record exists in the subjects table. If exists then delete the record by $id and store the success message in `session()->setFlashdata('message')` and class name in `session()->setFlashdata('alert-class')`,

Similarly, if a record is not deleted then store the failed message in `session()->setFlashdata('message'`) and class name in `session()->setFlashdata('alert-class')` .

Return to the `/` route.

```php
<?php namespace App\Controllers;

use App\Models\Subjects;

class SubjectsController extends BaseController
{

   public function index(){
      $subjects = new Subjects();

      ## Fetch all records
      $data['subjects'] = $subjects->findAll();
      return view('subjects/index',$data);
   }

   public function create(){
      return view('subjects/create');
   }

   public function store(){
      $request = service('request');
      $postData = $request->getPost();

      if(isset($postData['submit'])){

         ## Validation
         $input = $this->validate([
            'name' => 'required|min_length[3]',
            'description' => 'required'
         ]);

         if (!$input) {
            return redirect()->route('subjects/create')->withInput()->with('validation',$this->validator); 
         } else {

            $subjects = new Subjects();

            $data = [
               'name' => $postData['name'],
               'description' => $postData['description']
            ];

            ## Insert Record
            if($subjects->insert($data)){
               session()->setFlashdata('message', 'Added Successfully!');
               session()->setFlashdata('alert-class', 'alert-success');

               return redirect()->route('subjects/create'); 
            }else{
               session()->setFlashdata('message', 'Data not saved!');
               session()->setFlashdata('alert-class', 'alert-danger');

               return redirect()->route('subjects/create')->withInput(); 
            }

         }
      }

   }

   public function edit($id = 0){

      ## Select record by id
      $subjects = new Subjects();
      $subject = $subjects->find($id);

      $data['subject'] = $subject;
      return view('subjects/edit',$data);

   }

   public function update($id = 0){
      $request = service('request');
      $postData = $request->getPost();

      if(isset($postData['submit'])){

        ## Validation
        $input = $this->validate([
          'name' => 'required|min_length[3]',
          'description' => 'required'
        ]);

        if (!$input) {
           return redirect()->route('subjects/edit/'.$id)->withInput()->with('validation',$this->validator); 
        } else {

           $subjects = new Subjects();

           $data = [
              'name' => $postData['name'],
              'description' => $postData['description']
           ];

           ## Update record
           if($subjects->update($id,$data)){
              session()->setFlashdata('message', 'Updated Successfully!');
              session()->setFlashdata('alert-class', 'alert-success');

              return redirect()->route('/'); 
           }else{
              session()->setFlashdata('message', 'Data not saved!');
              session()->setFlashdata('alert-class', 'alert-danger');

              return redirect()->route('subjects/edit/'.$id)->withInput(); 
           }

        }
      }

   }

   public function delete($id=0){

      $subjects = new Subjects();

      ## Check record
      if($subjects->find($id)){

         ## Delete record
         $subjects->delete($id);

         session()->setFlashdata('message', 'Deleted Successfully!');
         session()->setFlashdata('alert-class', 'alert-success');
      }else{
         session()->setFlashdata('message', 'Record not found!');
         session()->setFlashdata('alert-class', 'alert-danger');
      }

      return redirect()->route('/');

   }
}
```

#
## View

Create a layouts and subjects folder at app/Views/.

Create the following files in the folders –

- layouts
  - [layout.php](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/app/Views/layouts/layout.php)
- subjects
  - [index.php](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/app/Views/subjects/index.php)
  - [create.php](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/app/Views/subjects/create.php)
  - [edit.php](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/app/Views/subjects/edit.php)

#
### Run

- Navigate to the project using Command Prompt if you are on Windows or terminal if you are on Mac or Linux, and
- Execute “php spark serve” command.

```
php spark serve
```

- Run `http://localhost:8080` in the web browser.

#
### Output

![](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/preview/example1.jpg) 

![](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/preview/example2.jpg) 

![](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/preview/example3.jpg) 

![](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/preview/example4.jpg) 

![](https://github.com/gzeinnumer/ci4_codeigniter4_crud_simple/blob/master/preview/example5.jpg) 

---

Thanks to [https://makitweb.com](https://makitweb.com/crud-create-read-update-delete-in-a-codeigniter-4/)

---

```
Copyright 2021 M. Fadli Zein
```