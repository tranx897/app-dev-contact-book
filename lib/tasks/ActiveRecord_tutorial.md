# DATABASE CREATION
``` (at terminal)
psql
CREATE DATABASE my_contact_book;
\connect my_contact_book
CREATE TABLE contacts (
  id SERIAL PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  date_of_birth DATE,
  street_address_1 TEXT,
  street_address_2 TEXT,
  city TEXT,
  state TEXT,
  zip TEXT,
  phone TEXT,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

# CONNECT DATABASE WITH RUBY APP
edit database in config/database.yml to database: my_contact_book

# CREATE ACTIVE RECORD CLASS TO INTERACT WITH DATABASE
rails c will open an IRB-like session in terminal
create a class Contact.rb in ./app/models so ActiveRecord can associate class name with your table
table attributes are not defined in the class. They were already defined when the table was created in the database
the class Contact only serves as a medium for our app to talk to the database

create a rake file in ./lib/taks for running large scripts with rake 
use Faker to generate fake data
```
task(:sample_contacts => :environment) do
  200.times do
    x = Contact.new
    x.first_name = Faker::Name.first_name
    x.last_name = Faker::Name.last_name
    x.save
  end
```

# Retrieving data
open a rails c session
`Contact.count` => number of records
`x = Contact.all` => an instance of ActiveRecord Relation (a table)
`x.sample` => sample record
`c = x[0]` => first record
`c.id` => id of first record
`c.first_name` => first name of first record

## Sorting
`Contact.all.order({ :last_name => :asc })` => order by last name ascending
`Contact.all.order({ :last_name => :asc, :first_name => :asc, :date_of_birth => :desc })` => tiebreaker with multiple sorting fields

## Where operations
`x = Contact.all.where({ :last_name => "Mouse" })` => filter by column
`x = Contact.where({ :last_name => "Mouse", :first_name => "Minnie" })` => returns a relation
`c = Contact.where({ :id => 2 }).at(0).first_name`  => first name accessible only after adding index of record (at 0)
`Contact.where({ :last_name => ["Betina", "Woods"] })` => multiple criteria for one column
`Contact.where({ :first_name => "Mickey" }).or(Contact.where({ :last_name => "Betina" }))` => OR search
`Contact.where({ :last_name => "Mouse" }).where({ :first_name => "Minnie" })` => Chaining multiple 'wheres' to narrow search
`Contact.where({ :last_name => "Mouse" }).where.not({ :first_name => "Mickey" })` => where not

### Fuzzy criteria
`Contact.where("last_name LIKE ?", "%bet%")` => % are wildcards, matche anything in that position
`Contact.where("date_of_birth > ?", 30.years.ago)`
`Contact.where("last_name >= ? AND last_name <= ?", "A", "C")`
`Instructor.where({ :last_name => ("A".."C") })`
```
start_date = 7.days.ago
end_date = Date.today
Contact.where({ :created_at => (start_date..end_date) })
```
two dots means inclusive of the second value, and three dots means exclusive of the second value. 
E.g., (1..4) is 1, 2, 3, and 4; (1...4) is only 1, 2, and 3.

## Limit and Offset
`Contact.where({ :last_name => "Mouse" }).limit(10)` => return no more than 10 records.
`Contact.where({ :last_name => "Mouse" }).offset(10).limit(10)` => helpful for second page of records

## Update
`c = Contact.where({ :id => 2 }).first` => locate the record first
`c.first_name = "Minerva"` => Then update to new value
`c.save` => Then save

## Delete
`c = Contact.where({ :id => 2 }).first`
`c.destroy`

## Min, Max, Average
`Contact.where({ :last_name => "Mouse" }).maximum(:date_of_birth)` => returns a single value of the max of the data column
`Contact.where({ :last_name => "Mouse" }).minimum(:date_of_birth)` 
`Review.where({ :venue_id => 4 }).average(:rating)`
