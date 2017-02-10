# Web Services

The web services package consists of 4 separate services/containers:

- api
- nginx
- doctorweb
- doctordb

## API


## nginx



## Doctor

Note that after creation, you should run the following command to setup the database:

```
docker exec doctor_web_1 bundle exec rake db:setup
```
