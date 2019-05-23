require 'octokit'
require 'faraday-http-cache'
require 'json'
require 'active_support/cache'
require 'active_support/cache/file_store'
require 'fileutils'
require 'csv'

Octokit.configure do |c|
  c.auto_paginate = true
end

Octokit.middleware = Faraday::RackBuilder.new do |builder|
  FileUtils.mkdir_p('cache')
  store = ActiveSupport::Cache::FileStore.new('cache')
  builder.use(Faraday::HttpCache, :serializer => Marshal, :shared_cache => false, :store => store)
  builder.use(Octokit::Response::RaiseError)
  builder.adapter(Faraday.default_adapter)
end

REPOS = [
  'docker',
  'powershell',
  'dsc',
  'acl',
  'amazon_aws',
  'azure_arm',
  'chocolatey',
  'reboot',
  'wsus_client',
  'dsc_lite',
  'scheduled_task',
  'kubernetes',
  'iis',
  'sqlserver',
  'registry',
  'helm',
  'rook',
  'stdlib',
  'apache',
  'apt',
  'firewall',
  'mysql',
  'ntp',
  'postgresql',
  'concat',
  'java',
  'tomcat',
  'vcsrepo',
  'inifile',
  'vsphere',
  'haproxy',
  'puppetlabs/cisco_ios',
  'websphere_application_server',
  'netscaler',
  'satellite_pe_tools',
  'exec',
  'ibm_installation_manager',
  'accounts',
  'java_ks',
  'motd',
  'tagmail',
  'bootstrap',
  'facter_task',
  'hocon',
  'package',
  'puppet_agent',
  'puppet_conf',
  'resource',
  'service',
  'translate',
  'yumrepo_core',
]

task :console do
  require 'pry'
  client = Octokit::Client.new(:access_token => ENV['GITHUB_ACCESS_TOKEN'])
  binding.pry
end

task :json, [:name] do |t, args|
  puts JSON.pretty_generate(search_repos(args[:name]))
end

task :csv, [:name] do |t, args|
  repo_data = search_repos(args[:name])

  summary_file = CSV.open('summary.csv', 'w', :headers => true)
  summary_file << ['repo', 'total_issues', 'open_issues', 'closed_issues', 'total_pulls', 'open_pulls', 'merged_pulls', 'closed_pulls']
  issues_file = CSV.open('issues.csv', 'w', :headers => true)
  issues_file << ['repo', 'id', 'title', 'state', 'user', 'user_type', 'created_at']
  pulls_file = CSV.open('pulls.csv', 'w', :headers => true)
  pulls_file << ['repo', 'id', 'title', 'state', 'user', 'user_type', 'created_at', 'merged_at']

  repo_data.each do |repo, data|
    summary_file << [
      repo,
      data[:summary][:issues][:total],
      data[:summary][:issues][:open],
      data[:summary][:issues][:closed],
      data[:summary][:pulls][:total],
      data[:summary][:pulls][:open],
      data[:summary][:pulls][:merged],
      data[:summary][:pulls][:closed],
    ]

    data[:pulls].each do |pull|
      pulls_file << [
        repo,
        pull[:id],
        pull[:title],
        pull[:state],
        pull[:user],
        pull[:user_type],
        pull[:created_at],
        pull[:merged_at],
      ]
    end

    data[:issues].each do |issue|
      issues_file << [
        repo,
        issue[:id],
        issue[:title],
        issue[:state],
        issue[:user],
        issue[:user_type],
        issue[:created_at],
      ]
    end
  end

  summary_file.close
  issues_file.close
  pulls_file.close
end

def search_repos(repos)
  repo_names = Array(repos || REPOS).map { |r| r.include?('/') ? r : "puppetlabs/puppetlabs-#{r}" }
  repo_names.map { |r| [r, search_repo(r)] }.to_h
end

def search_repo(repo)
  client = Octokit::Client.new(:access_token => ENV['GITHUB_ACCESS_TOKEN'])
  data = client.search_issues("repo:#{repo} created:>#{Time.now.utc.to_date - 90}")
  items = data.items.map { |r| parse_result_item(client, r) }
  pulls = items.select { |r| r[:type] == :pr }.map { |r| r.reject { |k, _| k == :type } }
  issues = items.select { |r| r[:type] == :issue }.map { |r| r.reject { |k, _| k == :type } }

  {
    :summary => {
      :pulls => {
        :total  => pulls.length,
        :open   => pulls.select { |r| r[:state] == 'open' }.length,
        :merged => pulls.select { |r| r[:state] == 'merged' }.length,
        :closed => pulls.select { |r| r[:state] == 'closed' }.length,
      },
      :issues => {
        :total  => issues.length,
        :open   => issues.select { |r| r[:state] == 'open' }.length,
        :closed => issues.select { |r| r[:state] == 'closed' }.length,
      },
    },
    :issues => issues,
    :pulls  => pulls,
  }
end

def parse_result_item(client, item)
  if item.pull_request?
    parse_pull_request(client, item.pull_request.rels[:self].get.data)
  else
    parse_issue(client, item)
  end
end

def parse_pull_request(_client, pr)
  result = {
    :type       => :pr,
    :user       => pr.user.login,
    :id         => pr.number,
    :title      => pr.title,
    :created_at => pr.created_at,
    :state      => pr.merged? ? 'merged' : pr.state,
    :user_type  => pr.author_association == 'NONE' ? 'contributor' : pr.author_association.downcase,
  }

  result[:merged_at] = pr.merged_at if pr.merged?

  result
end

def parse_issue(client, issue)
  {
    :type       => :issue,
    :user       => issue.user.login,
    :id         => issue.number,
    :title      => issue.title,
    :created_at => issue.created_at,
    :state      => issue.state,
    :user_type  => client.organization_member?('puppetlabs', issue.user.login) ? 'member' : 'contributor',
  }
end
