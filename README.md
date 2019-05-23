```
bundle install

# JSON report on all repos
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake repo > report.json

# JSON report on single repo
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake repo[puppetlabs/puppetlabs-stdlib]

# CSV report on all repos
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake csv

# CSV report on a single repo
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake csv[stdlib]
```
