# Criando uma API básica para consulta e venda de produtos

> Especificação REST em [RESTfulAPI.net](https://restfulapi.net)

## Requisitos:
- [PHP](https://secure.php.net/downloads.php)
- [Composer](https://getcomposer.org/download/)
- [Laravel](https://laravel.com/docs/5.6)

<!--  -->

## 1)   Modifique o arquivo de ambiente para que seja utilizado o [SQLite](https://laravel.com/docs/5.6/database) como banco de dados.
> - O SQLite é um mecanismo de banco de dados SQL autônomo, de alta confiabilidade, incorporado e repleto de recursos, de domínio público.
> [Mais...](https://www.sqlite.org/index.html)<br>
> - Ele foi utilizado para simplificar a configuração de ambiente.
> Para outros bancos de dados, consulte a [documentação](https://laravel.com/docs/5.6/database#introduction).

No arquivo `.env`, apague
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```
e coloque

```env
DB_CONNECTION=sqlite
```
> ATENÇÃO: O usuário que executar a aplicação precisa ter permissão de escrita no diretório [database](https://github.com/viniciuslj/exercicio-ce-ifes/tree/master/database).

<!--  -->

## 2)   Crie a tabela de Produtos.

- Crie o arquivo de migração utilizando o `artisan`:
    ```bash
    php artisan make:migration create_table_produtos
    ```

- Modifique o arquivo de migração.<br>
`database/migrations/*create_table_produtos.php`

    Adicione os campos da tabela no método `up`
    ```php
    Schema::create('produtos', function (Blueprint $table) {
        $table->increments('id');
        $table->string('nome', 100);
        $table->string('marca', 100);
        $table->decimal('preco', 10, 2);
        $table->timestamps();
    });
    ```

- Se não for utilizar autenticação de usuários, remova as migrações:
    `create_users_table`<br>
    `create_password_resets_table`<br>

    > O [Laravel](https://laravel.com) possui mecanismos prontos para facilitar autenticação. Consulte a documentação:
    > - [Authentication](https://laravel.com/docs/5.6/authentication)
    > - [Passport API Authentication](https://laravel.com/docs/5.6/passport)

- Crie o arquivo que será utilizado como banco de dados pelo SQLite.
    ```bash
    touch database/database.sqlite
    ```
- Execute a *migration* para criar a tabela.
    ```bash
    php artisan migrate
    ```

<!--  -->

## 3)   Crie o [Model](https://laravel.com/docs/5.6/eloquent#defining-models) para a tabela `produtos`
```bash
php artisan make:model Produto
```

<!--  -->

## 4)   Crie o [Controller]([documentação](https://laravel.com/docs/5.6/controllers)) para a API de produtos
```bash
php artisan make:controller API/ProdutoController --api
```

<!--  -->

## 5)   Configure as [Rotas](https://laravel.com/docs/5.6/routing)
Adicione o seguite código no arquivo de `routes/api.php`
```php
Route::apiResource('produtos', 'API\ProdutoController');
```
Essa linha adicionará as seguintes rotas na aplicação:
> Confira as rotas da sua aplicação da seguinte forma:<br>
> `php artisan route:list`

| Method    | URI                    | Name             | Action                                             | Middleware |
|-----------|------------------------|------------------|----------------------------------------------------|------------|
| GET\|HEAD | api/produtos           | produtos.index   | App\Http\Controllers\API\ProdutoController@index   | api        |
| POST      | api/produtos           | produtos.store   | App\Http\Controllers\API\ProdutoController@store   | api        |
| GET\|HEAD | api/produtos/{produto} | produtos.show    | App\Http\Controllers\API\ProdutoController@show    | api        |
| PUT\|PATCH| api/produtos/{produto} | produtos.update  | App\Http\Controllers\API\ProdutoController@update  | api        |
| DELETE    | api/produtos/{produto} | produtos.destroy | App\Http\Controllers\API\ProdutoController@destroy | api        |

<!--  -->

## 6)   Codifique os métodos da API de produtos no [ProdutoController](https://github.com/viniciuslj/exercicio-ce-ifes/blob/master/app/Http/Controllers/API/ProdutoController.php):

> RESTfulAPI [HTTP Methods](https://restfulapi.net/http-methods)

- Adicione a utilização de outras classes:
    ```php
    use Illuminate\Http\Request;
    use Illuminate\Http\Response;
    use App\Produto;
    use App\Http\Controllers\Controller;
    ```

- Listando todos os produtos [(HTTP GET)](https://restfulapi.net/http-methods/#get)
    ```php
    public function index()
    {
        $produtos = Produto::all();
        return response($produtos, Response::HTTP_OK);
    }
    ```
    > SQL equivalente: `SELECT * FROM produtos;`

- Criando um produto [(HTTP POST)](https://restfulapi.net/http-methods/#post)<br>
    *(Para atribuições de campos em massa, consulte [Mass Assignment](https://laravel.com/docs/5.6/eloquent#mass-assignment))*
    ```php
    public function store(Request $request)
    {
        $produto = new Produto();
        $produto->nome = $request->nome;
        $produto->marca = $request->marca;
        $produto->preco = $request->preco;
        $produto->save();
        return response($produto, Response::HTTP_CREATED);
    }
    ```
    > SQL equivalente: `INSERT INTO produtos(...) VALUES(...);`

- Exibindo apenas um produto [(HTTP GET)](https://restfulapi.net/http-methods/#get)<br>
    > O método `findOrFail` dispara uma exceção `ModelNotFoundException` caso o produto não seja localizado.<br>
    > Essa exceção retorna `HTTP_NOT_FOUND 404` em vez de `HTTP_OK 200`.
    ```php
    public function show($id)
    {
        $produto = Produto::findOrFail($id);
        return response($produto, Response::HTTP_OK);
    }
    ```
    > SQL equivalente: `SELECT * FROM produtos WHERE id = $id;`

- Alterando os dados de um produto [(HTTP PUT)](https://restfulapi.net/http-methods/#put)<br>
    *O produto atualizado é retornado.*
    ```php
    public function update(Request $request, $id)
    {
        $produto = Produto::findOrFail($id);
        $produto->nome = $request->nome;
        $produto->marca = $request->marca;
        $produto->preco = $request->preco;
        $produto->save();
        return response($produto, Response::HTTP_OK);
    }
    ```
    > SQL equivalente: `UPDATE produtos SET ... WHERE id = $id;`

- Removendo um produto [(HTTP DELETE)](https://restfulapi.net/http-methods/#delete)<br>
    *O produto removido é retornado.*
    ```php
    public function destroy($id)
    {
        $produto = Produto::findOrFail($id);
        $produto->delete();
        return response($produto, Response::HTTP_OK);
    }
    ```
    > SQL equivalente: `DELETE FROM produtos WHERE id = $id;`

<!--  -->

## 7)   Execute a aplicação em modo de testes:
```bash
php artisan serve
```

<!--  -->

## 8)   Execute testes no [Postman](https://www.getpostman.com)
Importe a collection disponível no arquivo `postman_collection_teste.json`

## License
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
    <img alt="Licença Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" />
</a>
<br/>
Este trabalho está licenciado com uma Licença 
<br/>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
    Creative Commons - Atribuição-NãoComercial 4.0 Internacional
</a>.
