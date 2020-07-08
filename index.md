# EthDNS and EthLink


## What is EthDNS?

[EthDNS](https://eips.ethereum.org/EIPS/eip-1185) is a way to access information in the [Ethereum Name Service](https://ens.domains/) (ENS) from DNS.


## What is EthLink?

EthLink is EthDNS for the `.eth` domain.  Because `.eth` is not a registered DNS top-level domain it is normally inaccessible from DNS, but by appending `.link` to the domain the relevant information cam be obtained.  For example, a DNS A record request for `mydomain.eth.link` would look up the A records in ENS for `mydomain.eth`.


## What information can be accessed from EthDNS?

DNS information held in ENS can be accessed in exactly the same way as traditional DNS.

## How to access EthDNS

EthDNS is operated as a DNS-over-HTTPS server. It uses HTTPS protocol to perform DNS resolution. Its endpoint is `https://eth.link/dns-query`.

It resolves regular DNS as well as ENS registered domains.

```json
$ curl -H 'content-type: application/dns-json' 'https://eth.link/dns-query?type=TXT&name=wealdtech.eth'
{
  "AD":true,"CD":false,"RA":true,"RD":true,"TC":false,"Status":0,
  "Question":[{"name":"wealdtech.eth.","type":16}],
  "Answer":[
  {"name":"wealdtech.eth","type":16,"TTL":3600,"data":"dnslink=/ipns/www.wealdtech.eth"},
  {"name":"wealdtech.eth","type":16,"TTL":3600,"data":"contenthash=0xe501017000117777772e7765616c64746563682e657468"},
  {"name":"wealdtech.eth","type":16,"TTL":3600,"data":"a=0x4760cF82331ee520573bbB332106353587E7eC49"}
  ]
}
```

To access it through standard DNS query, you need to setup a [DNS proxy](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/) to `https://eth.link/dns-query`. You can use [DNSCrypt-Proxy](https://github.com/DNSCrypt/dnscrypt-proxy/wiki/installation#setting-up-dnscrypt-proxy) or [cloudflared](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/).

## How does EthDNS work with IPFS?

ENS has a `contenthash` field which contains a pointer to content somewhere on the internet,  most commonly in IPFS.  If EthDNS is asked to serve a domain with a `contenthash` it will carry out the following operations:

- if the domain does not have an A record, when asked for an A record it will return that of an IPFS gateway
- when asked for TXT records it will return the contenthash both as its own field and as a DNSLink reference

For example:

```
$ dig @127.0.0.1 +short wealdtech.eth A  
104.18.64.168
$ dig @127.0.0.1 +short wealdtech.eth TXT
"dnslink=/ipns/www.wealdtech.eth"
"contenthash=0xe501017000117777772e7765616c64746563682e657468"
"a=0x4760cF82331ee520573bbB332106353587E7eC49"
```

This is used by eth.link to create a reverse proxy sitting in front of an IPFS Gateway.

The practical upshot of this is that if a user enters the URL `https://wealdtech.eth.link/` in to their web browser it will return IPFS content based on the  information within ENS without any changes required to their system  (browser plugins, alternate DNS servers, *etc.*).


## How do I add a content hash to my domain?

Content hashes can be managed at https://manager.ens.domains/

There are also [various libraries and CLI tools](https://docs.ens.domains/dapp-developer-guide/ens-libraries) that can manage content hashes.


## How do I add DNS information to my domain?

There are [various libraries and CLI tools](https://docs.ens.domains/dapp-developer-guide/ens-libraries) that can manage DNS information on ENS.


## Are domains other than `.eth` supported?

EthLink only applies to `.eth` domains, however other domains registered in ENS can be accessed directly when enabled.


## How do I enable ENS-based resolution of my DNS information?

Enabling ENS-based resolution is a simple case of using the EthLink. You should CNAME your domain to resolver.cloudflare-eth.com. TLS certificate is available on demand.

For instance, eth.link DNS record looks like

```
eth.link CNAME 3600 resolver.cloudflare-eth.com
```


## Further reading

[A Distributed Web Resolver for the rest of us](cloudflare blog)

[The basic workings of EthDNS, the system that enables DNS records on ENS.](http://www.wealdtech.com/articles/ethdns-an-ethereum-backend-for-the-domain-name-system/)

[Details of how DNSLink records are handled by IPFS gateways.](https://docs.ipfs.io/guides/concepts/dnslink/)