# Configuration:

- Set up host vars

    - In cloud_tools/ansible/host_vars:

        - there should already be a copy of research.local.yml named the same as the DNS name of the server you are managing, with no extension like ".yml" - just the hostname, nothing more.
        - verify that the server variables there match the server you are managing. You should only need to verify the following:

            - **`ansible_user`**: your user ID
            - **`server_host_name`**
            - **`server_IP_address`**

        - basic django setup:

            - **`django_project_name`**: to whatever you want your project to be named. Default is "research".
            - **`django_secret_key`**: Generate a new secret key for your application and store it here. On linux, you can make a random string of alphanumeric characters using "`openssl rand -base64 64`".

        - database settings:

            - if you are connecting to an existing postgresql database server with existing credentials, set:

                - **`django_database_type`** to "postgresql" (this is the default, so you don't need to do anything else for python packages, etc.).
                - **`django_db_host`** to database server host name.
                - **`django_db_name`** to database name.
                - **`django_db_username`** to your database user.
                - **`django_db_password`** to your database user's password.
                - **`django_db_create_db_user`** and **`django_db_create_database`** to **false**. so django configures the connection in `settings.py` but doesn't try to create the database or the user.

            - if you are connecting to postgresql and want a new database and/or user created for your django project, set everything above, but then set the variables **`django_db_create_db_user`** and **`django_db_create_database`** for what you want created to **true**.
            - FUTURE - if you set **`django_database_type`** to "sqlite3", this role will create a sqlite3 database file and do all migrations there. If you plan on connecting to an external database, but don't know all the details, do this.
