version: "3.7"
services:
  node:
    # For running on Aarch64 add `-aarch64` after `DATE`
    image: ghcr.io/subspace/node:gemini-1b-2022-june-05
    volumes:
# Instead of specifying volume (which will store data in `/var/lib/docker`), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
      - node-data:/var/subspace:rw
#      - /path/to/subspace-node:/var/subspace:rw
    ports:
# If port 30333 is already occupied by another Substrate-based node, replace all
# occurrences of `30333` in this file with another value
      - "0.0.0.0:30333:30333"
    restart: unless-stopped
    command: [
      "--chain", "gemini-1",
      "--base-path", "/var/subspace",
      "--execution", "wasm",
      "--pruning", "archive",
      "--port", "30333",
      "--rpc-cors", "all",
      "--rpc-methods", "safe",
      "--unsafe-ws-external",
      "--validator",
# Replace `INSERT_YOUR_ID` with your node ID (will be shown in telemetry)
      "--name", "suntao"
    ]
    healthcheck:
      timeout: 5s
# If node setup takes longer then expected, you want to increase `interval` and `retries` number.
      interval: 30s
      retries: 5

  farmer:
    depends_on:
      node:
        condition: service_healthy
    # For running on Aarch64 add `-aarch64` after `DATE`
    image: ghcr.io/subspace/farmer:gemini-1b-2022-june-05
# Un-comment following 2 lines to unlock farmer's RPC
#    ports:
#      - "127.0.0.1:9955:9955"
# Instead of specifying volume (which will store data in `/var/lib/docker`), you can
# alternatively specify path to the directory where files will be stored, just make
# sure everyone is allowed to write there
    volumes:
      - farmer-data:/var/subspace:rw
#      - /path/to/subspace-farmer:/var/subspace:rw
    restart: unless-stopped
    command: [
      "--base-path", "/var/subspace",
      "farm",
      "--node-rpc-url", "ws://node:9944",
      "--ws-server-listen-addr", "0.0.0.0:9955",
# Replace `WALLET_ADDRESS` with your Polkadot.js wallet address
      "--reward-address", "stB2LsNx1rcSfGAhMGtymsq7R4k6HhxuYCct7GSyXJJJNGB4b",
# Replace `PLOT_SIZE` with plot size in gigabytes or terabytes, for instance 100G or 2T (but leave at least 10G of disk space for node)
      "--plot-size", "1G"
    ]
volumes:
  node-data:
  farmer-data: