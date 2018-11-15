## Docker

```bash
# Run docker accessible via localhost:8888, which proxies to the Docker container proxy 2368
# -d detaches process to a background thread
docker run -d -p 8888:2368 ghost
# [2018-11-15 17:40:11] INFO Your blog is now available on http://localhost:2368/
```
## AWS

```bash
# Copy an S3 bucket (optional)
aws s3 sync s3://mybucket s3://backup-mybucket

# Remove objects from a bucket (in the example, all .log files)
aws s3 rm s3://mybucket/ --recursive --exclude "*" --include "*.log"
 
# Add SSL to Cloudfront (needs the 'file://' or it won't work!)
aws iam upload-server-certificate \
  --server-certificate-name server \
  --certificate-body file://server.crt \
  --private-key file://server.key \
  --certificate-chain file://server.ca-bundle \
  --path /cloudfront/
 
# If upload command doesn't work, can troubleshoot by attempting to upload certificate via AWS Console. Note: won't show up as a selectable CloudFront certificate because missing --path
# EC2 > Load Balancer > Select a balancer > Listeners tab > SSL Certificate > Change
# Public KEY, private CRT, chain CA-BUNDLE
 
# Remove certificate
aws iam delete-server-certificate --server-certificate-name my_cert
  
# Get cert info
aws iam list-server-certificates
```

## Git

```bash
# Squash many commits from a feature branch into a single commit to master
# Merge the feature branch into the master branch.
git merge feature_branch

# Reset the master branch to origin's state
# All files are unversioned, commit as you would normally in IDE
git reset origin/master
  
# Restore a point-in-time commit to a branch (for loading snapshots)
git branch new-branch-name <commit SHA from GitHub>
  
# Sync list of remote branches on local machine
git remote update origin
# Prune removed remote branches (i.e. archived)
git remote prune origin
# List remote branches
git branch -r
  
# Remove all commits after 'foo'
git reset --hard foo
 
# Merge tool
git mergetool --tool=opendiff
 
# Cherry pick hotfix from staging to prod. Can branch off master and generate a PR to be safe
git checkout master
git cherry-pick <commit hash from GitHub>
```

## Heroku

```bash
# Verify remote connectivity
heroku apps
   
# Add SSH key to Heroku (prompts if local key does not exist)
heroku keys:add ~/.ssh/id_rsa.pub
   
# Restart server
heroku restart --remote staging
  
# View processes (deployment takes approx. 3 minutes to switch over)
heroku ps --remote staging
# View logs
heroku logs --remote staging
heroku logs --tail --remote staging
   
# Turn off web servers by activating maintenance mode
heroku maintenance:on --remote staging
# Turn off worker processes by scaling to 0
heroku ps:scale worker=0 --remote staging
   
# Clear Rails cache (e.g. dashboard posts are cached)
# Boot the console
heroku run console --app my-app
  
# Slug over 300Mb and can't build
heroku repo:gc --app my-app
heroku repo:purge_cache --app my-app
 
# Running a rake task (i.e. scheduled job)
heroku run rake --trace db:migrate --app my-app
heroku run rake my_task[2010,1,2017,5,true] --app my-app
 
# Provisioning add-ons
heroku addons:create heroku-postgresql:hobby-basic --remote staging
 
# Copy prod database to staging (overwrite)
heroku pg:copy my-prod::HEROKU_POSTGRESQL_GRAY_URL HEROKU_POSTGRESQL_RED_URL -a my-staging
```

## Mongo

```bash
# Connect
mongo <MONGODB_URI>
 
# Query user documents that are not unread
{"target.id":28071,"context.status":{$ne:"unread"}}
 
# Remove documents from a collection (name is a key of actor)
db.dogs.remove({"actor.name":"Fido"})
# Other examples
db.dogs.remove({"context.breed":"hound"})
db.dogs.remove({"context.breed":"terrier"})
 
# Update documents from a collection (set all user 28071 documents to unread)
db.dogs.updateMany({"target.id":28071}, {$set: {"context.status":"fed"}})
 
# Export mLab database to the current folder
mongodump -h <db_host:port> -d <db_name> -u <user> -p <password>
 
# Import/Restore mLab database from dump files
mongorestore -h <db_host> -d <db_name> -u <user> -p <password> <dump_folder_path>
 
# Get the most recent document (from command line)
db.dogs.find().sort({_id:-1}).limit(1)
```

## Node

```bash
# Save to dev dependency in package.json
npm install package-name --save-dev

# List all outdated modules
npm outdated
```

## Postgres

```bash
# verify running
which psql
 
# if blank
sudo find / -name psql
# should get a result that looks similar to
# /Library/PostgreSQL/9.3/bin/psql
# /Library/PostgreSQL/9.3/pgAdmin3.app/Contents/SharedSupport/psql
open ~/.bash_profile
# add this line to bash_profile where $PATH is one of the valid paths minus psql.. See Example Below
export PATH=$PATH:/Library/PostgreSQL/9.3/bin/
 
# if error requiring libpqxx library..
brew install libpqxx
 
# Stopping and starting postgres (if not installed using Homebrew etc.)
vi ~/.bash_profile
# Add the following
alias postgres.server="sudo -u postgres pg_ctl -D/Library/PostgreSQL/9.2/data"
  
# Start/stop shortcut commands
postgres.server start
postgres.server stop
   
# If you get the error: fe_sendauth: no password supplied / FATAL:  password authentication failed
# Trust all local connects so no password needed
sudo vi /Library/PostgreSQL/9.4/data/pg_hba.conf
# change all host methods from 'md5' to 'trust' then stop/start postgres
 
# Create a new empty database named 'foo' (e.g. using Navicat)
# Restore to local database
pg_restore --verbose --clean --no-acl -U postgres -h localhost -d foo download.db
# If an error/warning count at bottom, simply re-run command
```

### Heroku

```bash
# List all backups on Heroku
heroku pg:backups --app my-app
   
# Create a backup (if limit reached, deletes the oldest existing backup). Limit seems to be 22.
heroku pg:backups capture --app my-app
  
# Get public URL of last backup
heroku pg:backups public-url --app my-app
# Get public URL of a specified backup (e.g. a001)
heroku pg:backups public-url a001 --app my-app
   
# Download the backup
Simply copy and paste the generated URL in a browser (curl produced an error) to download (e.g. download.db)
   
# Restore backup to Heroku instance
heroku pg:backups restore b101 DATABASE_URL --app my-app
```

### SQL

```sql
-- All table counts
SELECT relname, n_tup_ins - n_tup_del as rowcount FROM pg_stat_all_tables ORDER BY rowcount DESC
 
-- table row counts
create or replace function
count_rows(schema text, tablename text) returns integer
as
$body$
declare
  result integer;
  query varchar;
begin
  query := 'SELECT count(1) FROM ' || schema || '.' || tablename;
  execute query into result;
  return result;
end;
$body$
language plpgsql;
select
  table_schema,
  table_name,
  count_rows(table_schema, table_name)
from information_schema.tables
where
  table_schema not in ('pg_catalog', 'information_schema')
  and table_type='BASE TABLE'
order by 3 desc 
 
-- rollback statement
begin;
delete from foo where bar = 123;
rollback;
```

## Rails

```bash
# Generate a schema migration to add a new field to the posts table
rails g migration add_title_to_posts title:string
 
# Generate a new table with model
rails g model my_new_table foo:string, bar:string
 
# Run the migration locally
rake db:migrate
  
# Rollback the last migration
rake db:rollback
 
# Rollback to specific version
rake db:migrate:down VERSION=<version>
 
# Clear the cache (used by newsfeed etc.)
Rails.cache.clear
 
# Load schema to test database
rake db:setup RAILS_ENV=test
rake db:migrate RAILS_ENV=test
 
# Run all tests
rspec
bundle exec rspec
 
# Run a specific test
rspec spec/foo/foo_spec.rb
 
# Debugging a FactoryGirl setup issue
# Launch server in console (with dependencies)
rails c
# Manually create, which will show the SQL insert statements. Beware changes to FactoryGirl files require kill and restart "rails c"
FactoryGirl.create :dog, :with_bone
 
# Generate a Gem dependency report (finds outdated gems and gem vulnerabilities)
bundle exec gemsurance
```
