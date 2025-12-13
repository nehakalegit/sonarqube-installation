# sonarqube-installation
Sonarqube installation manually in ec2 amazon linux
Sonarqube to run requires atleast 2GB RAM - t2.small

Steps

//Install java-17
--sudo yum update -y
--sudo yum search java-17
--sudo yum install java-17-amazon-corretto -y
--java -version

//Install postgresql - Even though sonarqube server have default database, however this is not used in real production environment. New database is used using //postgresql(Sonaqube only supports PostgreSQL, Microsoft SQL Server (MSSQL), and Oracle)
--sudo dnf update -y
-- sudo dnf install search postgresql
--sudo dnf install postgresql-server postgresql -y // postgresql-server and postgresql these are two separate packages.postgresql 
// Postgresql - Client tools only-psql (CLI client), pg_dump, pg_restore, createdb, dropdb, Client libraries (libpq)
//Postgresl-server - The actual database server, postgres daemon(for setting username and password), initdb, Systemd service files, Default data directory setup
--sudo postgresql-setup --initdb
--sudo systemctl start postgresql
--sudo systemctl enable postgresql


