heroku-buildpack-snowflake-odbc
===

## Usage
This requires [heroku-buildpack-apt](https://github.com/heroku/heroku-buildpack-apt) to run before
it, with the following in the Aptfile:

```
unixodbc-dev
https://raw.githubusercontent.com/zzhaobraze/heroku-buildpack-snowflake-odbc/master/snowflake-odbc.deb
```
