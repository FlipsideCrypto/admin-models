## Profile Set Up

#### Use the following within profiles.yml

----

```yml
admin:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: <ACCOUNT>
      role: <ROLE>
      user: <USERNAME>
      password: <PASSWORD>
      region: <REGION>
      database: ADMIN_DEV
      warehouse: <WAREHOUSE>
      schema: silver
      threads: 4
      client_session_keep_alive: False
      query_tag: <TAG>
```

## Datashare Operations

### call_sp_grant_share_permissions

Executes overloaded stored procedure `sp_grant_share_permissions()`

```sh
# Applies grants to all databases
dbt run-operation call_sp_grant_share_permissions
# Applies grants to a single database
dbt run-operation call_sp_grant_share_permissions --args '{args: $$BSC$$}'
# Applies grants to all databases that have been deployed within the last hour
dbt run-operation call_sp_grant_share_permissions --args "{args: sysdate() - interval '1 hour'}"
# Applies grants to a database that have been deployed within the last hour with the default suffixes
dbt run-operation call_sp_grant_share_permissions --args '{args: "$$BSC$$, sysdate() - interval $$1 hour$$"}'
```

## Resources:

* Learn more about dbt [in the docs](https://docs.getdbt.com/docs/introduction)
* Check out [Discourse](https://discourse.getdbt.com/) for commonly asked questions and answers
* Join the [chat](https://community.getdbt.com/) on Slack for live discussions and support
* Find [dbt events](https://events.getdbt.com) near you
* Check out [the blog](https://blog.getdbt.com/) for the latest news on dbt's development and best practices

* Check out [the blog](https://blog.getdbt.com/) for the latest news on dbt's development and best practices

## Applying Model Tags

### Database / Schema level tags

Database and schema tags are applied via the `add_database_or_schema_tags` macro.  These tags are inherited by their downstream objects.  To add/modify tags call the appropriate tag set function within the macro.

```

{{ set_database_tag_value('SOME_DATABASE_TAG_KEY', 'SOME_DATABASE_TAG_VALUE') }}
{{ set_schema_tag_value('SOME_SCHEMA_TAG_KEY', 'SOME_SCHEMA_TAG_VALUE') }}

```

### Model tags

To add/update a model's snowflake tags, add/modify the `meta` model property under `config` .  Only table level tags are supported at this time via DBT.

```

{{ config(

    ...,
    meta={
        'database_tags':{
            'table': {
                'PURPOSE': 'SOME_PURPOSE'
            }
        }
    },
    ...

) }}

```

By default, model tags are not pushed to snowflake on each load.  You can push a tag update for a model by specifying the `UPDATE_SNOWFLAKE_TAGS` project variable during a run.

```

dbt run --var '{"UPDATE_SNOWFLAKE_TAGS": True}' -s models/core/core__fact_swaps.sql

```

### Querying for existing tags on a model in snowflake

```

select *
from table(admin.information_schema.tag_references('admin.core.fact_blocks', 'table'));
```

### Running streamline permissions from Snowflake

```
call admin.streamline.create_streamline_users_roles_dev('<PROD_DB_NAME>', '<AWS_LAMBDA_DEV_USER_PASSWORD>');

call admin.streamline.create_streamline_users_roles_prod('<PROD_DB_NAME>', '<AWS_LAMBDA_PROD_USER_PASSWORD>', 'DBT_CLOUD_PROD_USER_PASSWORD');

call admin.streamline.streamline_dev_permissions('<DEV_DB_NAME>');

call admin.streamline.streamline_prod_permissions('<PROD_DB_NAME>');
```