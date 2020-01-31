heroku-buildpack-snowflake-odbc
===

# Requirements
This requires [heroku-buildpack-apt](https://github.com/heroku/heroku-buildpack-apt) be installed
and ran before this.

## Installation
The order of the buildpack for Heroku is important. The following steps can be follow in the below order.
Or after the buildpacks are installed, the orders can be changed via the `Heroku Dashboard -> Settings`,
under Buildpack. Move the buildpacks so the `heroku-buildpack-apt` appears before the
`heroku-buildpack-snowflake-odbc`.

```
https://buildpack-registry.s3.amazonaws.com/buildpacks/heroku-community/apt.tgz
https://github.com/zzhaobraze/heroku-buildpack-snowflake-odbc
heroku/ruby
```


*The below assumes other required buildpacks are already installed*

### Add this buildpack first by running this [command](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app#adding-a-buildpack):

```
heroku buildpacks:add --index 1 https://github.com/zzhaobraze/heroku-buildpack-snowflake-odbc
```

### Add the apt [buildpack](https://github.com/heroku/heroku-buildpack-apt) next which will push it to the first order, and push this one down:
```
heroku buildpacks:add --index 1 https://buildpack-registry.s3.amazonaws.com/buildpacks/heroku-community/apt.tgz
```

### Aptfile
Add an [Aptfile](Appfile) to your Heroku root file with the following:
```
unixodbc-dev
https://raw.githubusercontent.com/zzhaobraze/heroku-buildpack-snowflake-odbc/master/snowflake-odbc-2.20.4.x86_64.deb
```

## Snowflake Connection
For ease of use, a Heroku ENV with the connection string can be used instead of a DSN file. The ENV variable ie `SNOWFLAKE_CONN_STR` will be in the following format:

```
SNOWFLAKE_CONN_STR:
	DRIVER=SnowflakeDSIIDriver;Locale=en-US;SERVER=[ACCOUNT].[REGION].snowflakecomputing.com;PORT=443;ACCOUNT=[ACCOUNT];DATABASE=[DATABASE];SCHEMA=[SCHEMA];WAREHOUSE=[WAREHOUSE];ROLE=[ROLE];SSL=on;QUERY_TIMEOUT=270;UID=[UID];PWD=[PWD];
```

## Snowflake Rails ODBC Setup
For usage in rails, the [Connect Rails to Snowflake by Localytics](https://eng.localytics.com/connecting-to-snowflake-with-ruby-on-rails/) tutorial can be used.

* [Snowflake ODBC Instructions](https://docs.snowflake.net/manuals/user-guide/odbc.html)

### Localytics ODBC_Adaptor
[Localytics ODBC_Adaptor](https://github.com/localytics/odbc_adapter)

Gemfile:
```
gem 'odbc_adapter'
```
or for newer version of Rails(5+) which updates [`bind_visitor` to `visitor`](https://github.com/localytics/odbc_adapter/pull/26) :
```
gem 'odbc_adapter', :git => 'https://github.com/zzhaobraze/odbc_adapter.git'
```

config/initializers/odbc.rb from Localytics:
```
require 'active_record/connection_adapters/odbc_adapter'
require 'odbc_adapter/adapters/postgresql_odbc_adapter'

ODBCAdapter.register(/snowflake/, ODBCAdapter::Adapters::PostgreSQLODBCAdapter) do
  # Explicitly turning off prepared statements as they are not yet working with
  # snowflake + the ODBC ActiveRecord adapter
  def prepared_statements
    false
  end

  # Quoting needs to be changed for snowflake
  def quote_column_name(name)
    name.to_s
  end

  private

  # Override dbms_type_cast to get the values encoded in UTF-8
  def dbms_type_cast(columns, values)
    values.each do |row|
      row.each_index do |idx|
        row[idx] = row[idx].force_encoding('UTF-8') if row[idx].is_a?(String)
      end
    end
  end
end
```

### Rails Configuration
config/database.yml:
```
snowflake:
  adapter: odbc
  conn_str: <%= ENV['SNOWFLAKE_CONN_STR'] %>
```

/config/application.yml:
```
SNOWFLAKE_CONN_STR: "DRIVER=SnowflakeDSIIDriver;......"
```

*Note, for local usage, driver maybe `Snowflake`*

model:
```
class [ModelName] < ApplicationRecord
  establish_connection(:snowflake)
end
```


### Rails Model
While using new version of Rails (ie 6.0.0), ActiveRecord doesn't seem to work well, so direct exec_query might have to be used:
```
results = [ModelName].connection.exec_query("select ...,.... from [Model_Name] where .....")
results.each do | result |
	...
end
```
*Note, be aware of potential SQL Injection*


## Reference
* [Connect Rails to Snowflake by Localytics](https://eng.localytics.com/connecting-to-snowflake-with-ruby-on-rails/)
* [Localytics ODBC_Adaptor](https://github.com/localytics/odbc_adapter)
* [heroku-buildpack-apt](https://github.com/heroku/heroku-buildpack-apt)
* [Buildpack Install](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app#adding-a-buildpack)
* [Snowflake ODBC Instructions](https://docs.snowflake.net/manuals/user-guide/odbc.html)