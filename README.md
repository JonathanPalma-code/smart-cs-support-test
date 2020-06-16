# Customer Service Support Technician Tech Test
Test for CS Support candidates

## [The Requirements](#requirements) | [My Notices & Recommendations](#recommendations)

## <a name="requirements">Requirements</a>

Allowing for setting up the project, this tech test should take around 2 hours to complete.

1. Find the cause of issues in the application:
* There are three bugs to find
* You are not required to *fix* the bugs, your aim is to __identify__ what is causing them

2. Document your process:
  * What did you notice/find?
  * What are your recommendations?

## Getting Started
These instructions will get the application up and running on your local machine for bug finding purposes.

* Install Ruby [Ways of Installing Ruby](https://www.ruby-lang.org/en/downloads)
* Install Yarn [Ways of Installing Yarn](https://yarnpkg.com/lang/en/docs/install)
* Clone this git repository `git clone git@github.com:smartpension/smart-cs-support-test.git`
* cd into the locally cloned repo `smart-cs-support-test`
* Run `bin/setup` 
* Start up your web-server (see the output of `bin/setup`)
 * If you get an error with: `getaddrinfo: nodename nor servname provided, or not known (SocketError)` 
   * Try running your server with`./bin/rails server -b 0.0.0.0`
     
* Navigate to: http://localhost:3000

Remember to document __how__ you identified the bugs and attach your findings to your email back to us, have fun!!

## <a name="recommendations">My Notices & Recommendations</a>

### Bug 1:

I was trying to POST/CREATE a new employee. It processed by EmployeesController#create as HTML, giving the error that the surname can't be blank. I found the bug thanks to 

*"employee"=>{"forename"=>"Jonathan", "middlename"=>"Palma"}*
 
**My recommendation is:**

In *app/models/employee.rb*, I took off the middlename, because you are trying to validate a data that there isn't any input for it (text_field) in *app/views/employees/new.html.erb*.

```
class Employee < ApplicationRecord
    belongs_to :company
    validates :forename, :surname, presence: true
end
```

In *app/views/employees/new.html.erb*, the text_field is assigned for middlename. It should be surname.

```
<p>
  <%= form.label :surname %>
  <%= form.text_field :surname, class: "form-control" %>
</p>
```

### Bug 2:

I've tried to submit/add a company without any input field/s and it allowed me to make that action. Yet, in this case, the web app shouldn’t allow it. 

**My recommendation is:** 

In *app/models/employee.rb*, add *validates :name, :details, presence: true*.

```
class Company < ApplicationRecord
    has_many :employees, dependent: :destroy
    accepts_nested_attributes_for :employees
    validates :name, :details, presence: true
end
```
 
with this, every time that a user wants to add/submit a new company, he will have to input data in the required fields; Name and details.

### Bug 3:

On the web app, I was trying to view all employees from a particular company and it was outputting only the employees of the first company. Even if I wanted to check all employees from the last company added into the database.

**My recommendation are:**

To check *app/views/companies/show.html.erb* and everything looked right on that file for me.

To check *app/controllers/companies_controller.rb* andI found something that shouldn’t be there. Otherwise, it would keep outputting the same event that I was experiencing before.

```
def show
    @company = Company.find(permitted_params[:id])
    @company = Company.first
end
```

*@company = Company.first* is basically saying: whenever they click a company, that is the second, third or fourth on the line, always show me the first company in line.

```
def show
    @company = Company.find(permitted_params[:id])
end
```

### Bug 4:

While trying to edit an employee on http://localhost:3000/companies/13/employees/4/edit, it was giving me an error saying that it *“Couldn't find Company with 'id'=13”*. 

The ID 13 is part of the Employee ID. To track which employee you want to edit, it is required the Company and Employee’s ID. 
I've started to check in *app/views/companies/show.html.erb* in tbody element we have:

```
<tbody>
    <% for employee in @company.employees %>
        <tr>
            <td><%= employee.id %></td>
            <td><%= employee.forename %></td>
            <td><%= employee.surname %></td>
            <td><%= link_to "Edit", edit_company_employee_path(employee), class: "btn btn-primary btn-sm" %></td>
        </tr>
    <% end %>
</tbody>
```

As the method is telling you to do *edit_company_employee_path( ... )*.

**My recommendation is:**

to add on that line *@company*:

```
<td><%= link_to "Edit", edit_company_employee_path(@company, employee), class: "btn btn-primary btn-sm" %></td> 
```

so it can identify from which company you want to update the employee's data.
While navegating, the address won't be the same, because http://localhost:3000/companies/4/employees/13/edit:
* Company ID ⇒ 4
* Employee ID ⇒ 13
