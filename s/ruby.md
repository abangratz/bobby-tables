Ruby
====

Whether you are using [ActiveRecord][0], DataMapper[1], Sequel[2] or any other ORM, do avoid the following:

Example 1: Interpolation

    sql = "SELECT * FROM users WHERE username = '#{params[:username]}'"
    # ... execute the previous statement

Example 2: sprintf-style

    sql = "UPDATE users SET password = %s where login = %s" % [pass, login]
    # ... execute the previous statement

This will effectively avoid quoting and make your code prone to SQL injection. But there are ways to get it right
for the most common ORMs:

ActiveRecord
------------

To avoid the trap mentioned at the beginnig, you could do:

    Person.find_by_sql ['SELECT * from persons WHERE name = ?', name]

Also, if you use the ``where`` method, you should never use the following:

    Person.where("name LIKE '%#{params[:name]}'")

as it will set you up for SQL injection. You should rather use:

    Person.where("name LIKE ?", params[:name])

or the placeholder syntax:

    Person.where("name LIKE :name", {name: params[:name]})

where you name the positions and assign the values via hash.

A third and preferrable method is to use hashes for querying if checking for equality, range conditions or subsets.

    Person.where(name: params[:name])
    Person.where(dob: (5.years.ago)...(1.day.ago))
    Person.where(eyecolor: %w[blue green])

DataMapper
----------

Similar to ActiveRecord, DataMapper provides you with functions, but relieves you of the need to use snippets
altogether:

    Person.where(:name.like => "%#{params[:name]}")
    Person.where(:age.gt => 15)

The variables are automatically quoted correctly.

Retrieving a record via id should be best done via:

    Person.get(params[:id].to_i)

Sequel
------

Sequel provides you with an interface that does not patch symbols, but makes quoting mostly implicit:

    Person.filter(Sequel.like(:name, "%#{params[:name]}%"))
    Person.filter { age > 5 }

There's also support for snippets and placeholders:

    Person.filter("name like ?", "%#{params[:name]}%")
    Person.filter("age > :min_age", {min_age: 5})

Finding a record by id is best done via:

    Person[params[:id].to_i]


Ruby/DBI
--------

Using [Ruby/DBI](http://ruby-dbi.rubyforge.org/) is analog to [Perl](./perl.html).

Sidenote: most of the time it is better to use an ORM, they come as lightweight as you need them and relieve you from a
lot of stress regarding quoting.


Summary
-------

Some of the previous examples show a bit of rails boilerplate code (namely the ``params[...]`` syntax), but as rails is
the most uses framework out there, it is quite representative. Developers using anything else are quite probably aware
of ways to sanitize input.

It's best to avoid constructing query strings via interpolation. It is far better to rely on the methods provided by the
ORMs.  Most of those methods are quite uniform. It is also a good idea to convert the parameters into the format that is
corresponding to the database column for sanitizing the input.


[0]: http://guides.rubyonrails.org/active_record_querying.html
[1]: http://datamapper.org/docs/find.html
[2]: http://sequel.rubyforge.org/rdoc/files/doc/dataset_filtering_rdoc.html
