# Bolsa de Trabajo - Ruby on Rails

## Objetivo

Construir un página web donde se pueda publicar y **filtrar** por tipo de ofertas de trabajo.

---

## Mini Magick

En este repositorio se está haciendo uso de la gema [mini_magick](https://github.com/minimagick/minimagick).
Para poder utilizar sin problemas debemos tener instalado `ImageMagick` or `GraphicsMagick`.


#### Windows

Para instalar GraphicsMagick en Windows puedes seguir los pasos indicados [aquí](http://www.graphicsmagick.org/INSTALL-windows.html)

#### Mac

Para instalar en OSX se puede utilizar el siguiente comando. Se requiere tener Homebrew previamente instalado.

```
brew install graphicsmagick
```

ó

```
brew install imagemagick

```
#### Ubuntu

```
sudo apt-get install imagemagick
```

#### Heorku

Para desplegar la aplicación en Heroku se debe utilizar este [buildpack](https://elements.heroku.com/buildpacks/xerpa/heroku-buildpack-graphicsmagick)

---

## Modelos

Los principales modelos que vamos a utilizar en este proyecto son `User` y `Job`


### Jobs

Cada `Job`  va a tener los siguientes atributos:

- Title -  `title:`
- Description - `description:text`
- Logo Avatar - via [Carrierwave](https://github.com/carrierwaveuploader/carrierwave) - `avatar:string`
- Website URL - `url:string`
- Type: Fulltime, Part-Time, Freelance, Contract `job_type:string`
- Location - `location:string`
- User ID - `user_id:integer`
- Remote ok - `remote_ok: boolean, default: false`
- apply_url - `apply_url:string`

Each `User` will have:

  - Name - `name:string`
  - Email - `email:string`
  - Admin - `admin:boolean`


### Crear el proyecto <a name="paso1"></a>

  ```ruby
  rails new Jobs
  cd Jobs
  ```

### Scaffold <a name="paso2"></a>

```ruby
$ rails g scaffold Job title:string description:text url:string job_type:string location:string remote_ok:boolean apply_url:string
```

Notemos que no hemos considerado `avatar` ni `user_id`. Agregaremos estos luego donde avatar será `string` y user_id `integer`

No utilizamos un scaffold para `User` en esta aplicaión puesto que [Device](https://github.com/heartcombo/devise) nos ayudará con esta tarea. Device es una solución flexible para autenticación usando Rails.


Tenemos que ejecutar la migración puesto que hemos creado un nuevo modelo.

```ruby
rake db:migrate
```

### Seeds

Usaremos el archivo `db/seeds.rb` para crear datos para probar la aplicación.


```ruby
rake db:seed
```

> `rake db:seeds` carga la data de `db/seeds.rb` en nuestra base de datos.


### Iniciamos el servidor

```ruby
rails s
```



### Validar admin user

La columna `admin` nos va a indicar si un usuario es admin o no. En todas nuestras vistas o `views` podemos utilizar un método para verificar si un usuario es admin o no. El método `current_user` es un helper que nos provee [Device](https://github.com/heartcombo/devise)

```ruby
if current_user.try(:admin?)
 # show something only admins can see here
end
```

En el archivo `index.html.erb`

```ruby
<% if current_user.try(:admin) || job_author(job) %>
  <ul class="pv3">
    <li class="inline-block f6">Admin controls: </li>
    <li class="inline-block">
      <%= link_to 'View', job, class: 'button is-small is-link is-outlined' %></li>
    <li class="inline-block">
      <%= link_to 'Edit', edit_job_path(job), class: 'button is-small is-link is-outlined' %></li>
    <li class="inline-block">
      <%= link_to 'Delete', job, method: :delete, data: { confirm: 'Are you sure?' }, class: 'button is-small is-link is-outlined' %></li>
  </ul>
<% end %>
```

El helper `job_author` nos indica si el usuario ha iniciado sesión y si el trabajo posteado le corresponde a este usuario.
Para más detalle ver `jobs_helper.rb`
### Agregar filtro

Vamos a crear un `partial` para nuestro panel donde listaremos los tipos de trabajos que tenemos.
Creamos el archivo `_panel.html.erb`

```html
<nav class="panel">
  <p class="text-uppercase has-text-gray pv1 f6">filter by job type</p>

  <ul class="mb3 pa0">
    <li class="pv1">
      <%= link_to 'All', jobs_path, class: "button is-fullwidth is-dark" %>
    </li>

    <li class="pv1">
      <%= link_to 'Full-time', jobs_path(job_type: "Full-time"), class: "button is-fullwidth is-primary" %>
    </li>

    <li class="pv1">
      <%= link_to 'Part-time', jobs_path(job_type: "Part-time"), class: "button is-fullwidth is-link" %>
    </li>

    <li class="pv1">
      <%= link_to 'Freelance', jobs_path(job_type: "Freelance"), class: "button is-fullwidth is-warning" %>
    </li>

    <li class="pv1">
      <%= link_to 'Contract', jobs_path(job_type: "Contract"), class: "button is-fullwidth is-info" %>
    </li>
  </ul>
</nav>
```

En el archivo `index.html.erb` agregamos nuestro partial

```html
<div class="column is-2 bg-light">
  <div class="pl3 pr1">
    <%= render 'panel' %>
  </div>
</div>
```

En nuestro controlador `jobs_controller.rb`
retornamos el listado de todos los trabajos publicados

```ruby
@jobs = Job.all.order("created_at desc") # SELECT * FROM jobs
```

### Query customizada con `find_by_sql`

Ahora vamos a filtrar por tipo de trabajo.
Nos vamos a `app/controllers/jobs_controller.rb`

En Jobs Controller, llamamos el método `find_by_sql` en la clase Job. Este método va a ejecutar una query SQL válido. Como resultado, obtenemos los atributos listados en la cláusula `select`.


```ruby
@jobs = Job.find_by_sql("SELECT * FROM jobs WHERE job_type = '"+params[:job_type]+"'")
```

También podemos usar el siguiente comando para realizar el mismo query

```ruby
@jobs = Job.where(job_type: params[:job_type]).order("created_at desc")
```


> Notemos que no es necesario tener la definición de un modelo para que `find_by_sql` funcione. Este método solo ejecuta un query SQL query contra la base de datos; el query no se preocupe de tener una clase Active Record model.



### Siguientes pasos

Hasta el momento hemos utilizado filtros basados en atributos para realizar filtrados sencillos. Si bien esto es bastante útil en nuestras aplicaciones web, a veces necesitamos realizar `queries` más complejas en nuestra base de datos.

Tenemos la relación `uno a muchos` entre `User` y `Job`. Es decir, un usuario puede tener muchas ofertas de trabajo publicadas.

Qué otros queries puedes utilizar para enriquecer la aplicación?


### Más información

* Documentación [find_by_sql](https://api.rubyonrails.org/classes/ActiveRecord/Querying.html)
* Documentación de [find_by_sql](https://apidock.com/rails/ActiveRecord/Querying/find_by_sql)
* Ejemplo de cómo usar [find_by_sql](http://underpop.online.fr/r/ruby/rails/tutorial/ruby-on-rails-3-13.html)
* Más ejemplos de[SQL query con Ruby](https://edgeguides.rubyonrails.org/active_record_querying.html)
