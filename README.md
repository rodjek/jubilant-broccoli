```
bundle install

# Report on all repos
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake repo > report.json

# Report on single repo
GITHUB_ACCESS_TOKEN=<github oauth token> bundle exec rake repo[puppetlabs/puppetlabs-stdlib]
```
