# Setup

Load the `pg_tde` at the start time. The extension requires additional shared memory; therefore,  add the `pg_tde` value for the `shared_preload_libraries` parameter and restart the `postgresql` instance.

1. Use the [ALTER SYSTEM](https://www.postgresql.org/docs/current/sql-altersystem.html) command from `psql` terminal to modify the `shared_preload_libraries` parameter.

    ```sql
    ALTER SYSTEM SET shared_preload_libraries = 'pg_tde';
    ```

2. Start or restart the `postgresql` instance to apply the changes.

    * On Debian and Ubuntu:    

       ```sh
       sudo systemctl restart postgresql.service
       ```
    
    * On RHEL and derivatives

       ```sh
       sudo systemctl restart postgresql-16
       ```

3. Create the extension using the [CREATE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html) command. You must have the privileges of a superuser or a database owner to use this command. Connect to `psql` as a superuser for a database and run the following command:

    ```sql
    CREATE EXTENSION pg_tde;
    ```
    
    By default, the `pg_tde` extension is created for the currently used database. To enable data encryption in other databases, you must explicitly run the `CREATE EXTENSION` command against them. 

    !!! tip

        You can have the `pg_tde` extension automatically enabled for every newly created database. Modify the template `template1` database as follows: 

        ```
        psql -d template1 -c 'CREATE EXTENSION pg_tde;'
        ```

4. Setup a key provider for the database

    * With keyring file:    

       ```sql
       SELECT pg_tde_add_key_provider_file('provider-name','/path/to/the/keyring/data.file');
       ```

       This setup is intended for development, and stores the keys unencrypted in the specified data file.

    * With keyring vault

       ```sql
       SELECT pg_tde_add_key_provider_vault_v2('provider-name',:'secret_token','url','mount','ca_path');
       ```
       
       where:

       * `url` is the URL of the Vault server
       * `mount` is the mount point where the keyring should store the keys
       * `secret_token` is an access token with read and write access to the above mount point
       * [optional] `ca_path` is the path of the CA file used for SSL verification

5. Add a master key

        ```sql
       SELECT pg_tde_set_master_key('name-of-the-master-key', 'provider-name');
       ```
