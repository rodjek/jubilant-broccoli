require 'octokit'
require 'faraday-http-cache'
require 'json'
require 'active_support/cache'
require 'active_support/cache/file_store'
require 'fileutils'

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


task :repo, [:name] do |t, args|
  repos = Array(args[:name] || REPOS)
  repo_names = repos.map { |r| r.include?('/') ? r : "puppetlabs/puppetlabs-#{r}" }
  puts JSON.pretty_generate(repo_names.map { |r| [r, search_repo(r)] }.to_h)
end

def search_repo(repo)
  client = Octokit::Client.new(:access_token => ENV['GITHUB_ACCESS_TOKEN'])
  data = client.search_issues("repo:#{repo} created:>#{Time.now.utc.to_date - 90}")
  items = data.items.map { |r| parse_result_item(r) }
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

def parse_result_item(item)
  if item.pull_request?
    parse_pull_request(item.pull_request.rels[:self].get.data)
  else
    parse_issue(item)
  end
end

def parse_pull_request(pr)
  result = {
    :type       => :pr,
    :user       => pr.user.login,
    :id         => pr.number,
    :title      => pr.title,
    :created_at => pr.created_at,
    :state      => pr.merged? ? 'merged' : pr.state,
  }

  result[:merged_at] = pr.merged_at if pr.merged?

  result
end

def parse_issue(issue)
  {
    :type       => :issue,
    :user       => issue.user.login,
    :id         => issue.number,
    :title      => issue.title,
    :created_at => issue.created_at,
    :state      => issue.state,
  }
end
