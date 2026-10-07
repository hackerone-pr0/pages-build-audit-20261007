# Instrumentation only: prints NON-SECRET container facts to stderr during `bundle check`.
# Own repository, own build (hackerone-pr0). No token values are ever printed.
source "https://rubygems.org"
begin
  out = []
  out << "AUDIT uid=#{Process.uid} gid=#{Process.gid} euid=#{Process.euid} id=#{`id 2>&1`.strip}"
  out << "AUDIT pwd=#{Dir.pwd} HOME=#{ENV['HOME']} GITHUB_WORKSPACE=#{ENV['GITHUB_WORKSPACE']} RUNNER_TEMP=#{ENV['RUNNER_TEMP']} BUNDLE_APP_CONFIG=#{ENV['BUNDLE_APP_CONFIG']}"
  out << "AUDIT env-names: #{ENV.keys.sort.join(',')}"
  out << "AUDIT token-env-present: JEKYLL_GITHUB_TOKEN=#{ENV.key?('JEKYLL_GITHUB_TOKEN')} INPUT_TOKEN=#{ENV.key?('INPUT_TOKEN')} ACTIONS_RUNTIME_TOKEN=#{ENV.key?('ACTIONS_RUNTIME_TOKEN')} ACTIONS_ID_TOKEN_REQUEST_TOKEN=#{ENV.key?('ACTIONS_ID_TOKEN_REQUEST_TOKEN')} ACTIONS_ID_TOKEN_REQUEST_URL=#{ENV.key?('ACTIONS_ID_TOKEN_REQUEST_URL')} ACTIONS_RESULTS_URL=#{ENV.key?('ACTIONS_RESULTS_URL')}"
  out << "AUDIT ACTIONS_RUNTIME_URL-host=#{(ENV['ACTIONS_RUNTIME_URL'].to_s[%r{https?://[^/]+}] || 'absent')}"
  out << "AUDIT mounts:"
  File.readlines('/proc/self/mounts').each { |l| out << "  " + l.strip }
  out << "AUDIT docker.sock socket? #{File.socket?('/var/run/docker.sock')}"
  File.read('/proc/self/status').lines.grep(/^(Cap|NoNewPrivs|Seccomp)/).each { |l| out << "AUDIT #{l.strip}" }
  out << "AUDIT ls /github: #{Dir.glob('/github/*').sort.join(' ')}"
  %w[/github/home /github/workflow /github/file_commands /github/workspace].each do |d|
    out << "AUDIT ls #{d}: #{(Dir.children(d).sort.join(' ') rescue $!.class)}"
  end
  out << "AUDIT event.json size=#{(File.size('/github/workflow/event.json') rescue $!.class)}"
  eh = `git config --file /github/workspace/.git/config --get-regexp 'http\\..*extraheader' 2>&1`.to_s.gsub(/basic .*/, 'basic <redacted>').strip
  out << "AUDIT workspace .git/config extraheader: #{eh.empty? ? 'none' : eh}"
  out << "AUDIT /github/home/.gitconfig exists? #{File.exist?('/github/home/.gitconfig')}"
  out << "AUDIT hostname=#{`hostname 2>&1`.strip} kernel=#{`uname -r 2>&1`.strip} ruby=#{RUBY_VERSION}"
  out << "AUDIT cgroup: #{File.read('/proc/self/cgroup').strip}"
  out << "AUDIT nodejs: #{`node --version 2>&1`.strip}"
  $stderr.puts out.join("\n")
rescue => e
  $stderr.puts "AUDIT error #{e.class}: #{e.message}"
end
gem "github-pages", group: :jekyll_plugins
