# portScannerLite.js
portScannerLite.js A lightweight TCP port scanner that asynchronously checks a target host for common open ports (FTP, SSH, Telnet, SMTP, DNS, HTTP, HTTPS, RDP). Uses Node's built-in net module with socket timeouts to determine port status, demonstrating network reconnaissance fundamentals. Intended for authorized testing only.
// portScannerLite.js
// Checks whether common TCP ports are open on a target host.
// Educational use only — scan only hosts you own or have permission to test.

const net = require("net");

const COMMON_PORTS = {
  21: "FTP", 22: "SSH", 23: "Telnet", 25: "SMTP",
  53: "DNS", 80: "HTTP", 443: "HTTPS", 3389: "RDP",
};

function checkPort(host, port, timeout = 1000) {
  return new Promise((resolve) => {
    const socket = new net.Socket();
    socket.setTimeout(timeout);

    socket.on("connect", () => {
      socket.destroy();
      resolve({ port, open: true });
    });
    socket.on("timeout", () => {
      socket.destroy();
      resolve({ port, open: false });
    });
    socket.on("error", () => {
      resolve({ port, open: false });
    });

    socket.connect(port, host);
  });
}

async function scan(host) {
  console.log(`Scanning ${host}...\n`);
  const results = await Promise.all(
    Object.keys(COMMON_PORTS).map((p) => checkPort(host, parseInt(p)))
  );

  results.forEach(({ port, open }) => {
    const status = open ? "OPEN" : "closed";
    console.log(`Port ${port} (${COMMON_PORTS[port]}): ${status}`);
  });
}

const target = process.argv[2];
if (!target) {
  console.log("Usage: node portScannerLite.js <host>");
  console.log("Example: node portScannerLite.js scanme.nmap.org");
} else {
  scan(target);
}
