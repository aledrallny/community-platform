#Como introducir cambios en la base de datos

Para manejar migraciones de datos usamos una herramienta de migración Postgrator. Puede encontrar más información sobre el asunto en: https://github.com/rickbergfalk/postgrator

Each cp-*-service which uses postgresql has a directory `migrations` in `scripts/database/pg` directory, which will always contain a `001.do.init-db.sql` file
where lies the initial version of the schema. 

The 001.do.init-db.sql will run at system setup and will create your database initial schema.

Si necesita hacer cambios en el esquema de la base de datos puede hacer un archivo sql de migración del siguiente modo:

* Cree otro archivo *.sql  bajo el directorio  `migrations` . 

* El nobre del archivo debe seguir la siguente convención: [version].[action].[optional-description].sql See the details bellow: 

	* La Version debe ser un número, pero puede incrementar los números del modo que usted lo desee.

	* Action must be "do" (we won't support rollbacks, so "undo" is not an option for action)

	* Optional-description can be a label or tag to help keep track of what happens inside the script. Descriptions should not contain periods.

Aquí hay un ejemplo:

* crear un archivo  `002.do.add-test-column.sql` en el directorio `scripts/database/pg/migrations`;

* el script contiene lo siguiente:

```sql
DO $$ 
	BEGIN
		BEGIN
			ALTER TABLE cd_dojos ADD COLUMN test_column1 character varying;
		EXCEPTION
			WHEN duplicate_column THEN RAISE NOTICE 'column test_column1 already exists in cd_dojos.';
		END;
		BEGIN
			ALTER TABLE cd_dojos ADD COLUMN test_column2 character varying;
		EXCEPTION
			WHEN duplicate_column THEN RAISE NOTICE 'column test_column2 already exists in cd_dojos.';
		END;
	END;
$$
```

* Este script agrega dos nuevas columnas, solo si no existen ya, de otro modo desplegará un mensaje de error diciendo que las columnas ya están allí.

* to apply this changes to the DB schema you have to run the migration script in that service, like this:

`./start.sh development scripts/migrate-psql-db.js`

* this applies only the changes that haven't been applied yet to the schema (see the Postgrator documentation for more informations).
