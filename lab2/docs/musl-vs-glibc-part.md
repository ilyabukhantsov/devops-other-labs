# musl-vs-glibc-part

```
musl-vs-glibc-part git:(main) ✗ sudo sudo docker run --rm -it --name dns-server --network dns-lab \
alpine sh -c "apk add dnsmasq && \
echo 'address=/myservice.internal.corp/10.0.0.50' > /etc/dnsmasq.conf && \
dnsmasq -k --log-queries --log-facility=-"
(1/2) Installing dnsmasq-common (2.91-r0)
(2/2) Installing dnsmasq (2.91-r0)
  Executing dnsmasq-2.91-r0.pre-install
Executing busybox-1.37.0-r30.trigger
OK: 8682 KiB in 18 packages
dnsmasq[1]: started, version 2.91 cachesize 150
dnsmasq[1]: compile time options: IPv6 GNU-getopt no-DBus no-UBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP no-conntrack ipset no-nftset auth no-DNSSEC loop-detect inotify dumpfile
dnsmasq[1]: reading /etc/resolv.conf
dnsmasq[1]: using nameserver 127.0.0.11#53
dnsmasq[1]: read /etc/hosts - 9 names
dnsmasq[1]: query[AAAA] myservice.internal from 172.18.0.3
dnsmasq[1]: forwarded myservice.internal to 127.0.0.11
dnsmasq[1]: reply myservice.internal is NXDOMAIN
dnsmasq[1]: query[AAAA] myservice.internal.corp from 172.18.0.3
dnsmasq[1]: forwarded myservice.internal.corp to 127.0.0.11
dnsmasq[1]: reply myservice.internal.corp is NODATA-IPv6
dnsmasq[1]: query[A] myservice.internal from 172.18.0.3
dnsmasq[1]: cached myservice.internal is NXDOMAIN
dnsmasq[1]: query[A] myservice.internal.corp from 172.18.0.3
dnsmasq[1]: config myservice.internal.corp is 10.0.0.50
dnsmasq[1]: query[AAAA] myservice.internal from 172.18.0.3
dnsmasq[1]: cached myservice.internal is NXDOMAIN
dnsmasq[1]: query[A] myservice.internal from 172.18.0.3
dnsmasq[1]: cached myservice.internal is NXDOMAIN
dnsmasq[1]: query[AAAA] myservice.internal from 172.18.0.3
dnsmasq[1]: cached myservice.internal is NXDOMAIN
dnsmasq[1]: query[A] myservice.internal from 172.18.0.3
dnsmasq[1]: cached myservice.internal is NXDOMAIN
```

Результати експерименту показують, що DNS резолвінг у контейнеризованому середовищі залежить не лише від DNS сервера (dnsmasq), але й від клієнтського стеку резолвінгу та системної бібліотеки (glibc або musl). DNS сервер коректно обробляє запити та повертає очікувані результати для повних доменних імен, однак клієнтські запити можуть відрізнятися через механізм search domains та особливості NSS resolver.

Додатково, кешування NXDOMAIN та використання вбудованого DNS Docker (127.0.0.11) можуть впливати на поведінку резолвінгу, ускладнюючи діагностику проблем.