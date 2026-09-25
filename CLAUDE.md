# docker-atm11

Unraid Docker image for the All the Mods 11 server (CurseForge project 1148445). Adapted from `adamzaidi/docker-atmons`; the layout matches it, so fixes usually apply to both.

- `launch.sh` pins `SERVER_VERSION` + `SERVER_FILE_ID` (the ServerFiles zip, not the client pack file). `update-mod.yml` runs `fetch_latest_server_files.js` daily to bump them; a push to main builds and pushes `DOCKERHUB_USERNAME/docker-atm11:<version>` + `:latest`.
- ATM11 is Minecraft 26.1.2 / NeoForge and needs **Java 25** (its `user_jvm_args.txt` uses `-XX:+UseCompactObjectHeaders`, which Java 21 rejects).
- The pack's `user_jvm_args.txt` has no trailing newline. `launch.sh` adds one before appending `JVM_OPTS`, otherwise flags fuse together and the JVM won't start.
- The pack's `startserver.sh` installs NeoForge only when `libraries/` is missing, so the upgrade path wipes `libraries` along with config/kubejs/local/mods.
- `launch.sh` calls the pack's `startserver.sh` only with `ATM11_INSTALL_ONLY=true` (installs NeoForge and creates server.properties), then runs Java itself. startserver.sh as PID 1 ignored SIGTERM, so `docker stop` killed the server without saving. Now a SIGTERM trap writes `stop` to a FIFO on the server's stdin and waits.
- server.properties settings use `set_prop` (sets the key, or appends it if missing), because they have to apply on first boot, before Minecraft writes the full file.
- Restarts after a crash are left to Docker's restart policy (the pack's internal restart loop is no longer used).
- ForgeCDN URL = `files/<first 4 digits of id>/<last 3 digits, leading zeros stripped>/<name>`.
