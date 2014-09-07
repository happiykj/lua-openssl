lua-openssl toolkit - A free, MIT-licensed OpenSSL binding for Lua (Work in progress).

#Index

1. [Introduction](#1-introduction)
2. [Message Digest](#2-message-digest)
3. [Cipher](#3-cipher)
4. [Public/Private Key](#4-publicprivate-key-functions)
5. [Certificate](#5-certificate)
6. [PKCS7/CMS](#6-pkcs7cms)
7. [Pkcs12](#6-pkcs12-function)
8. [SSL](#8-ssl)
9. [Misc](#9-misc-functions)
+ A [Howto](#a-howto)
+ B [Examples](#b-example-usage)
+ C [Contact](#c-contact)

#1. Introduction

I needed a full OpenSSL binding for Lua, after googled, I couldn't find a version to fit my needs.
I found the PHP openssl binding is a good implementation, and it inspired me.
So I decided to write this OpenSSL toolkit for Lua.

The goal is to fully support the listed items.Below you can find the development progress of lua-openssl. 

* Symmetrical encrypt/decrypt. (Finished)
* Message digest. (Finished)
* Asymmetrical encrypt/decrypt/sign/verify/seal/open. (Finished)
* X509 certificate. (Finished)
* PKCS7/CMS. (Developing)
* SSL/TLS. (Finished)

Most of the lua-openssl functions require a key or certificate as argument; to make things easy to use OpenSSL,
This rule allow you to specify certificates or keys in the following ways:

1. As an openssl.x509 object returned from openssl.x509.read
2. As an openssl.evp_pkey object return from openssl.pkey.read or openssl.pkey.new

Similarly, you can also specify a public key as a key object returned from x509:get_public().

## lua-openssl modules
digest,cipher, x509, cms and so on, be write as modules.

```lua
   local digest = require'openssl'digest
   local cipher = require'openssl'cipher
   local crypto = require'crypto'
```

digest() equals with digest.digest(), same cipher() equals with cipher.cipher().

**crypto** is a compat module with [LuaCrypto](https://github.com/mkottman/luacrypto),
document should to [reference](http://mkottman.github.io/luacrypto/manual.html#reference)

**ssl** is a compat module with [luasec](https://github.com/brunoos/luasec),
document should to [refrence](https://github.com/brunoos/luasec/wiki/LuaSec-0.5).
NYI list: conn:settimeout,...


## lua-openssl Objects

The following are some important lua-openssl object types:

```
	openssl.bio,
	openssl.x509,
	openssl.stack_of_x509,
	openssl.x509_req,
	openssl.evp_pkey,
	openssl.evp_digest,
	openssl.evp_cipher,
	openssl.engine,
	openssl.pkcs7,
	openssl.cms,
	openssl.evp_cipher_ctx,
	openssl.evp_digest_ctx
	...
```

They are shortened as bio, x509, sk_x509, csr, pkey, digest, cipher,
	engine, cipher_ctx, and digest_ctx.

Please note that in the next sections of this document:

```
   => means return of a lua-openssl object
   -> means return of a basic Lua type (eg, string, boolean, etc)
```


If a function returns nil, it will be followed by an error number and string.

## Version

This lua-openssl toolkit works with Lua 5.1 or 5.2, and OpenSSL (0.9.8 or above 1.0.0). 
It is recommended to use the most up-to-date OpenSSL version because of the recent security fixes.

If you want to get the lua-openssl and OpenSSL versions from a Lua script, here is how:

```lua
openssl = require "openssl"
lua_openssl_version, lua_version, openssl_version = openssl.version()
```

## Bugs

Lua-Openssl is heavily updated, if you find bug, please report to [here](https://github.com/zhaozg/lua-openssl/issues/)

#2. Message Digest

* ***openssl.digest.list*** ([boolean alias=true])  -> array
 * Return all md methods default with alias

* ***openssl.digest.get***(string alg|int alg_id|openssl.asn1_object obj) => openssl.evp_digest
 * Return a evp_digest object

* ***openssl.digest.new*** (string alg|int alg_id|openssl.asn1_object obj) => openssl.evp_digest_ctx
 * Return a evp_digest_ctx object

* ***openssl.digest*** (string alg|openssl.evp_digest obj, string msg [,boolean raw=false]) -> string
 * Return a hash value for msg, if raw is true, it will be hex encoded

* ***evp_digest:new*** ([openssl.engine e]) => openssl.evp_digest_ctx
 * Return an evp_digest_ctx object

* ***evp_digest:info*** () -> table
 * Return a table with key nid, name, size, block_size, pkey_type, and flags

* ***evp_digest:digest*** (string msg[, openssl.engine e]) -> string
 * Return a binary hash value for msg

* ***digest_ctx:info*** () -> table
 * Return a table with key block_size, size, type and digest

* ***digest_ctx:update*** (string data) -> boolean

* ***digest_ctx:final*** ([string last [,boolean raw=true]) -> string
 * Return a hash value,default is binaray

* ***digest_ctx:reset*** ()
 * Cleanup evp_message_ctx to make it reusable.

* ***openssl.hmac*** (digest md, string msg,string key[, engine e=nil]) -> string
 * Return a hash value for msg, if raw is true, it will be hex encoded

* ***openssl.hmac.hmac*** (digest md, string msg,string key[, boolean raw=false[,engine e=nil]]) -> string
 * Return a hash value for msg, if raw is true, it will be hex encoded

* ***openssl.hmac.digest*** just a alias of ***openssl.hmac.hmac***

* ***openssl.hmac.new*** (digest md, string key[, engine e=nil]) => openssl.hmac_ctx
 * Return a hmac_ctx object
 
* ***hmac_ctx:update*** (string data)
 
* ***hmac_ctx:final*** ([string last[,boolean raw=false]]) -> string
 * Return a hmac value

* ***hmac_ctx:reset*** ()

#3. Cipher

* ***openssl.cipher.list*** ([boolean alias=true])  -> array
 *  return all cipher methods default with alias

* ***openssl.cipher.get*** (string alg|int alg_id|asn1_object obj) => evp_cipher
 *  return a evp_cipher method object

* ***openssl.cipher.encrypt***(string alg|int alg_id|asn1_object obj, string input_msg,
   string key [,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * return encrypted message,defualt with pad but without iv

* ***openssl.cipher.decrypt***(string alg|int alg_id|asn1_object obj, string input_msg,
 string key[,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * return decrypt message

* ***openssl.cipher.cipher***(string alg|int alg_id|asn1_object obj, boolean encrypt,string input_msg,
  string key[,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * return encrypted or decrypted message

* ***openssl.cipher***(string alg|int alg_id|openssl.asn1_object obj,boolean encrypt,string input_msg,
  string key[,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * return encrypted or decrypted message, this API implicated with metatable `__call` function.

* ***openssl.cipher.new***(string alg|int alg_id|openssl.asn1_object obj, boolean encrypt,
  string key[,string iv[,boolean pad=true[,openssl.engine e]]]) => openssl.evp_cipher_ctx
 * return evp_cipher_ctx object to encrypt or decrypt

* ***openssl.cipher.encrypt_new***(string alg|int alg_id|openssl.asn1_object obj,
  string key[,string iv[,boolean pad=true[,openssl.engine e]]]) => openssl.evp_cipher_ctx
 * return evp_cipher_ctx object to encrypt

* ***openssl.cipher.decrypt_new***(string alg|int alg_id|openssl.asn1_object obj,
   string key[,string iv[,boolean pad=true[,openssl.engine e]]]) => openssl.evp_cipher_ctx
 * return evp_cipher_ctx object to encrypt

* ***evp_cipher:info***() ->table
 * return table result with name, block_size,key_length,iv_length,flags,mode keys

* ***evp_cipher:BytesToKey***(string data[, string salt=nil [evp_digest|string md='sha1']}) -> string key,string iv
 * raturn key and iv according to input bytes

* ***evp_cipher:encrypt***(string input_msg, string key
  [,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * encrypt input_msg with key and return result

* ***evp_cipher:decrypt***(string input_msg, string key
 [,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * decrypt input_msg with key and return result

* ***evp_cipher:cipher***(boolean encrypt, string msg, string key,
 [,string iv[,boolean pad=true[,openssl.engine e]]]) -> string
 * encrypt or decrypt input msg and return result

* ***evp_cipher:new***(boolean encrypt, string key[,string iv[,boolean pad=true[,openssl.engine e]]]) => openssl.evp_cipher_ctx
 * create encrypt or decrypt evp_cipher_ctx object with key,iv ...., and return it

* ***evp_cipher:encrypt_new***(string key[,string iv[,boolean pad=true[,openssl.engine e]]])  => openssl.evp_cipher_ctx
 * create encrypt evp_cipher_ctx object, and return it

* ***evp_cipher:decrypt_new***(string key[,string iv[,boolean pad=true[,openssl.engine e]]])  => openssl.evp_cipher_ctx
 * create decrypt evp_cipher_ctx object, and return it

* ***cipher_ctx:info***() ->table
  * return table with block_size,key_length,iv_length,flags,mode,nid,type and evp_cipher(object) keys

* ***cipher_ctx:update***(string data) -> string
  * return result string, may be 0 length

* ***cipher_ctx:final()*** -> string
  * return result string, may be 0 length

#4. Public/Private key functions

* ***openssl.pkey.new*** ([string alg='rsa' [,int bits=1024|512, [...]]]) => evp_pkey
 * generate new keypair, support alg include rsa,dsa,dh,ec
 * default alg is RSA key, bits=1024, 3rd argument e default is 0x10001
 * dsa,with bits default 1024 ,and option seed data
 * dh, with bits(prime_len) default 512,
 * ec, with ec_name, and option flag

* ***openssl.pkey.new*** ({alg='rsa|dsa|dh|ec', ... }) => evp_pkey
 * contruct or generate a keypair object
 * private key should has it factor named n,q,e and so on, value is hex encoded string
 * The pattern pkey.new need table as pkey argument, and key 'alg' must be given.

```
 when arg is rsa, table may with key n,e,d,p,q,dmp1,dmq1,iqmp,both are string value
 when arg is dsa, table may with key p,q,g,priv_key,pub_key,both are string value
 when arg is dh, table may with key p,g,priv_key,pub_key,both are string value
 when arg is ec, table may with D,X,Y,Z,both are string value
```

* ***openssl.pkey.get_public***(evp_pkey private) => evp_pkey
 * Return public key for private key

* ***openssl.pkey.read*** (string|bio data [,bool public_key=false[,string fmt='auto [,string passphrase]]]}) => evp_pkey
 * Read from string or bio data,  return a EVP_PKEY object.
  It can be:

  1. data can be string or bio
  2. set public_key to true will load as public key.
  3. format support 'auto','pem' or 'der'
  4. if read private key, you may need to #4arg  passphrase 

### About padding

  Currently supports 6 padding modes. They are: pkcs1, sslv23, no, oaep, x931, pss.

* ***openssl.pkey.sign*** (evp_pkey key, string data [, evp_digest md|string md_alg=SHA1]) ->string
 * Uses key to create signature for data, returns signed result

* ***openssl.pkey.verify*** (evp_pkey key, string data, string signature [, evp_digest md|string md_alg=SHA1]) ->boolean
 * Uses key to verify that the signature is correct for the given data.

* ***openssl.pkey.encrypt*** (evp_pkey key, string data [,string padding=pkcs1]) -> string
 * Use key to encrypt data, default padding use 'pkcs1', data length should not longer than key size.

* ***openssl.pkey.decrypt*** (evp_pkey key, string data [,string padding=pkcs1]) -> string
 * Use key to decrypt data, default padding use 'pkcs1', data length must equals with key size.

* ***openssl.pkey.seal***(table pubkeys, string data[, cipher enc|string alg='RC4']) -> string,table|ekey,iv
 * Encrypts data using pubkeys in table, so that only owners of the respective private keys and ekeys can decrypt and read the data.
 * Returns the sealed data and table containing encrypted keys, hold envelope keys on success, else nil.

* ***openssl.pkey.seal***(evp_pkey pkey, string data[, cipher enc|string alg='RC4']) -> string,table
 * Encrypts data using pubkeys, so that only owners of the respective private keys and ekeys can decrypt and read the data.
 * Return sealed data,and encrypt key success, else nil.

* ***openssl.pkey.open*** (evp_pkey key, string data, string ekey,string iv[, evp_cipher enc|string md_alg=RC4]) -> string
 * Open/decrypt sealed data using private key,and the corresponding envelope key.
 * Returns decrypted data on success and nil on failure.

* ***evp_pkey:export*** ([boolean only_public = false [,boolean raw_key=false [,boolean pem=true,[, string passphrase]]]]) -> string
 * If only_public is true, will export public key
 * If raw_key is true, will export rsa, dsa or dh data
 * If passphrase exists, export key will be encrypted with it

* ***evp_peky:parse*** (evp_pkey key) -> table
 * Returns a table with the key details (bits, pkey, and type)
 * pkey may be rsa, dh, dsa, shown as table with factor hex encoded bignum.

* ***evp_pkey:is_private*** () -> boolean
 * Check whether the supplied key is a private key (by checking if the secret prime factors are set)

* ***evp_pkey:compute_key***(string remote_public_key) -> string
 * Compute shared secret for remote public key and local private key,Only for DH key.

* ***evp_pkey:encrypt*** (string data [,string padding=pkcs1]) -> string

* ***evp_pkey:decrypt*** (string data [,string padding=pkcs1]) -> string

* ***evp_pkey:sign***(string data[,digest md|string alg='sha1']) -> string

* ***evp_pkey:verify***(string data,string sig[,digest md|string alg='sha1']) -> boolean

* ***evp_pkey:seal***(string data[, cipher enc|string alg='RC4']) -> string,string,string

* ***evp_pkey.open***(string data, string ekey,string iv [, evp_cipher enc|string md_alg=RC4]) -> string


#5. Certificate

##X509 certificate

* ***openssl.x509.read*** (string cert|bio cert [,format='auto']) => x509
 * Return openssl.x509 object
 * cert is a string or bio object containing the data from the certificate file
 * format support 'auto', 'der' or 'pem', default with 'auto'

* ***x509:export*** ([string format='pem'[,bool notext=true]]) -> string
 * Export x509 as certificate content data
 * format only 'pem'(default) or 'der'

* ***x509:parse*** ([bool shortnames=true]) -> table
 * returns a table which contains all x509 information

* ***x509:get_public*** () => evp_pkey

* ***x509:check*** (evp_pkey pkey) -> boolean
 * Return true if  private key match with cert

* ***x509:check*** (sk_x509 ca [,sk_x509 untrusted[,string purpose]])->boolean
 * purpose can be one of: ssl_client, ssl_server, ns_ssl_server, smime_sign, smime_encrypt, crl_sign, any, ocsp_helper, timestamp_sign
 * ca is an openssl.stack_of_x509 object contain certchain.
 * untrusted is an openssl.stack_of_x509 object containing a bunch of certs that are not trusted but may be useful in validating the certificate.

***openssl.stack_of_x509*** is an important object in lua-openssl, it can be used as a certchain, trusted CA files or unstrust certs.

* ***openssl.sk_x509_read*** (string|bio certs_in_pems) => sk_x509

* ***openssl.sk_x509_new*** ([table array={}]} =>sk_x509

* ***sk_x509:push*** (openssl.x509 cert) => sk_x509

* ***sk_x509:pop*** () => x509

* ***sk_x509:set*** (number i, x509 cert) =>sk_x509

* ***sk_x509:get*** (number i) => x509

* ***sk_x509:insert*** (x509 cert,number i, ) =>sk_x509

* ***sk_x509:delete*** (number i) => x509

* ***sk_x509:totable*** () -> table
 * table as array contain x509 from index 1

* ***sk_x509:sort*** ()

* ***`#sk_x509`***
 * returns the number of certs in stack_of_x509

##certificate sign request

* ***openssl.csr.new*** (evp_pkey privkey, table dn[, table attribs[,table extensions[,string digest='sha1WithRSAEncryption'|digest md]]]]) => x509_req
 * Generates CSR with gived private key, dn, attribs and extensions

* ***openssl.csr.read*** (string data|BIO in,[string format='auto']) => x509_req
 * format support 'auto','pem','der',default use 'auto'

* ***csr:export*** ([string format='pem' [, boolean noext=true]])->string

* ***csr:get_public*** () -> evp_pkey

* ***csr:parse***([boolean shortname=true]) -> table

* ***csr:sign*** (x509 cacert=nil, evp_pkey caprivkey, table arg={serialNumber=,num_days=,version=,}[,table extensions]) => x509
 * args must have serialNumber as hexecoded string, num_days as number

##Certificate revocked list

* ***openssl.crl.read*** (string data) => x509_crl

* ***openssl.crl.new*** (x509 cacert [,table revoked={{sn=,revoketime=,reason=0},{}},[number lastUpdate[, number nextUpdate[,number version]]]]) => x509_crl
 * Create a new X509 CRL object, lastUpdate and nextUpdate is time_t value,
 * Revoked is a table on which hex cert serial is key, and time_t revocked time.

* ***crl:export***[string format='pem' [, boolean noext=true]] -> string

* ***crl:parse***([boolean shortname=true]) -> table

* ***crl:verify*** (x509 cacert) -> boolean

* ***crl:sign*** (evp_pkey privkey, [string alg=sha1WithRSAEncryption|digest md]) -> boolean

* ***crl:sort*** ()

* ***crl:set_version*** (number version=0) -> boolean

* ***crl:set_update_time*** ([number lastUpdate=date() [, number nextUpdate=last+7d]]) -> boolean

* ***crl:set_issuer*** (x509 cacert) -> boolean

* ***crl:add*** (string|number|bn serial, number revoketime [, string reason|number reason = 0]}) -> boolean

* ***crl:parse*** ([boolean shortname=true]) -> table
 * Below you can find an example of table content.

```
{
 sig_alg=sha1WithRSAEncryption
 lastUpdate=110627103001Z
 issuer={
     CN=CA
     C=CN
 }
 nextUpdate=110707110000Z
 nextUpdate_time_t=1310032800
 lastUpdate_time_t=1309167001
 crl_number=1008
 version=1
 hash=258a7571
 revoked={
     1={
         time=1225869543
         serial=4130323030303030303030314632
         reason=Superseded
     },
     ...
 }
}
```

##timestamp

lua-openssl timestamp modules has four object, ts_req,ts_resp,ts_resp_ctx,ts_verify_ctx

* ***openssl.ts.req_new***(string req, string|digest md
  [,table option={version=1,policy=,nonce=,cert_req=}] ) => openssl.ts_req

* ***openssl.ts.req_d2i***(string der) => openssl.ts_req

* ***openssl.ts.resp_d2i***(string der) => openssl.ts_resp

* ***openssl.ts.resp_ctx_new***(x509 tscert, evp_pkey tspkey, sk_x509 extra_certs,string default_policy,table options[, function serial_cb]) => ts_resp_ctx

* ***openssl.ts.verify_ctx_new***() => ts_verify_ctx

* ***ts_req:i2d***() ->string

* ***ts_req:parse***() -> table
 * Returns table with key status_info,token and tst_info

* ***ts_req:to_verify_ctx***() => ts_verify_ctx

* ***ts_resp:i2d***() ->string

* ***ts_resp:parse***() -> table
 * Returns table with key version,cert_req and msg_imprint

* ***ts_req:tst_info***() => table

* ***ts_resp_ctx:sign***(string req_der) => openssl.ts_resp

* ***ts_resp_ctx:sign***(ts_req obj) => openssl.ts_resp

* ***ts_verify_ctx:verify_response***(openssl.ts_resp obj) -> boolean

* ***ts_verify_ctx:verify_token***() -> boolean

#6. PKCS7/CMS

## PKCS7
* ***openssl.pkcs7.read***(bio|string in[,string format='auto']) => openssl.pkcs7,string
 * Read string or bio object, which include pkcs7 content, if success will return pkcs7 object or nil
 * format allow "auto","der","pem","smime", default is "auto" will try all method until load ok
 * when format is "smime", second return maybe content,if include.

* ***openssl.pkcs7.sign***(string|bio msg, x509 signcert, evp_pkey signkey[, int flags [,stack_of_x509 extracerts]]) => openssl.pkcs7
 * Signs message with signcert/signkey and return signed pkcs7 object.

* ***openssl.pkcs7.verify*** (pkcs7 in[, stack_of_x509 signerscerts [, stack_of_x509 cacerts,[, string|bio msg[,int flag]]]])
  ->string|boolean,openssl.sk_x509
 * Verify signed pkcs7 object, the signer is who they say they are, and returns the verify result,follow by signers.

* ***openssl.pkcs7.encrypt*** (string|bio msg, stack_of_x509 recipcerts, [, evp_cipher cipher[,string flags ]]) => openssl.pkcs7
 * Encrypts message with the certificates in recipcerts and output the return pkcs7 object.

* ***openssl.pkcs7.decrypt*** (pkcs7 in, x509 recipcert [,evp_pkey recipkey]) -> string
 * Decrypt pkcs7 message

* ***pkcs7:verify*** ([string flags [, stack_of_x509 signerscerts [, stack_of_x509 cacerts,
   [, stack_of_x509 extracerts [,bio content]]]}]) ->boolean, openssl.sk_x509
 * Verify signed pkcs7 object, the signer is who they say they are, and returns the verify result,follow by signers.

* ***pkcs7:decrypt*** (pkcs7 in, x509 recipcert [,evp_pkey recipkey]) -> string
 * Decrypt pkcs7 message

* ***pkcs7:export*** (boolean pem) -> string
 * export pkcs7 as a string, which can be to read

* ***pkcs7:parse***() -> table
 * Return a table has pkcs7 infomation, include type,and other things relate to types

## CMS
CMS are based on apps/cms.c from the OpenSSL dist, so for more information,
see the documentation for OpenSSL.

### API flags(string)

```
  "detached",
  "nodetached",
  "text",
  "nointern",
  "noverify",
  "nochain",
  "nocerts",
  "noattr",
  "binary",
  "nosigs"
```

Decrypts the S/MIME message in the BIO object and output the results to BIO object. 
recipcert is a CERT for one of the recipients. recipkey specifies the private key matching recipcert.

Headers is an array of headers to prepend to the message, they will not be included in the encoded section.

* ***openssl.cms.create***(...) => cms object
 * none paramater will create new cms
 * bio in[, number flags=0] will return data cms object
 * bio in, digest alg[, number flags=0] will return digest cms object
  
* ***openssl.cms.encrypt***(sk_x509 encerts,bio in,cipher cipher,int flags[, table options]) =>cms object
 * options may have key,keyid,password field,which must be string type  

* ***openssl.cms.decrypt***(cms cms,pkey pkey, x509 recipt, bio dcout,bio out, int flags[,table options]) -> boolean
 * options may have key,keyid,password field,which must be string type  

* ***openssl.cms.sign***(x509 cert, evp_pkey pkey, bio data[,number flags=0]) => cms object

* ***openssl.cms.verify***(cms obj,string verify_mode,...) -> boolean result
 * verify_mode must be verify,digest,receipt, default is verify.
 * 'verify' must followed by sk_x509, bio in, bio out, option flag (default is 0)
 * 'digest' must followed by bio in, bio out and option flags
 * 'receipt' must followed by source cms object, sk_x509 certs, x509_store and option falgs

* ***openssl.cms.read***(bio in[, string format='auto'[,...]]) => cms object
 * format support auto,smime,der,pem
 * if format is 'smime', need supply extra paramater bio object which have content

* ***openssl.cms.write***(cms obj, bio out, bio in[,number flags=0[,string fmt='smime']]) -> boolean

* ***openssl.cms.compress***(bio in[,string alg=zlib|rle[,int flags=0]])=> cms object

* ***openssl.cms.uncompress***(cms obj,bio in,bio out[,int flags=0]) -> boolean result

* ***openssl.cms.EncryptedData_encrypt***(bio in, cipher alg, string key[,number flags=0]) => cms object

* ***openssl.cms.EncryptedData_decrypt***(cms obj, string key, bio out[,number flags=0]) -> boolean result

#7. PKCS12 Function

* ***openssl.pkcs12.read*** (string pkcs12data, string pass) -> table
 * Parses a PKCS12 data to a table
 * Returns a table containing cert, pkey, and extracerts keys

* ***openssl.pkcs12.export*** (x509 cert, evp_pkey pkey, string pass [[, string friendname ], table extracerts])) -> string
 * Creates and exports a PKCS12 data
 * friendname is optional, if supplied, must be as 4th parameter
 * extracerts is optional, it can be as 4th or 5th parameter (if friendname is supplied)

#8. SSL

* ***openssl.ssl.ctx_new***(string SSL_protocol[, string support_ciphers]) => ssl_ctx
 * ssl_protocol can be SSLv3,SSLv23,SSLv2,TLSv1,DTLSv1, and can be follow by -server or -client
 * If not given support_ciphers, default of openssl will be used.

* ***openssl.ssl.alert_string***(number alter[,boolean long = false]) -> string, string
 * Return alter type string and alter desc string, if long set to long will return long info

## ssl_ctx Object

* ***ssl_ctx:use***(evp_pkey pkey,x509 cert) -> boolean[,string errmsg[,number errval]]
 * Tell ssl_ctx use private key and certificate, and check private key
 * Return true for ok, or return nil, follow by errmsg and errval

* ***ssl_ctx:add***(x509 clientca[,table extra_chain_cert_array]) -> boolean
 * Add client ca cert and option extra chian cert

* ***ssl_ctx:mode***([boolean clear=nil] string mode ...) -> string ...
 * If clear set true, given mode list will be clear, or will be set
 * Return new mode list
 * mode support: 

```
 'enable_partial_write',
 'accept_moving_write_buffer',
 'auto_retry',
 'no_auto_chain',
 'release_buffers'
```
  
  eg:

```
 modes = { ssl_ctx:mode('enable_partial_write','accept_moving_write_buffer','auto_retry') },
 
 for  i, v in ipairs(modes)
  print(v)
 end

 #output 'enable_partial_write','accept_moving_write_buffer','auto_retry'
```

* ***ssl_ctx:options***([boolean clear=nil] string options ...) -> string ...
 * If clear set true, given option list will be clear, or will be set
 * Return new options list
 * option support: 

```
	"microsoft_sess_id_bug",
	"netscape_challenge_bug",
	"netscape_reuse_cipher_change_bug",
	"sslref2_reuse_cert_type_bug",
	"microsoft_big_sslv3_buffer",
	"msie_sslv3_rsa_padding",
	"ssleay_080_client_dh_bug",
	"tls_d5_bug",
	"tls_block_padding_bug",
	"dont_insert_empty_fragments",
	"all",
```

* ***ssl_ctx:timeout***([number timeout]) -> number
 * If not give arg timeout, will return current, or use new timeout value and return previous.

* ***ssl_ctx:verify_mode***() -> ...
 * return mode list, please see below

* ***ssl_ctx:verify_mode***(...) 
 * args must be in "none", "peer", "fail", "once"
 * you can pass more than one mode at same time

## SSL object

* ***ssl_ctx:quiet_shutdown***([boolean mode]) -> [boolean]
Normally when a SSL connection is finished, the parties must send out
"close notify" alert messages using ***SSL:shutdown"*** for a clean shutdown.

When setting the "quiet shutdown" flag to 1, ***SSL:shutdown*** will set the internal flags
to SSL_SENT_SHUTDOWN|SSL_RECEIVED_SHUTDOWN. ***SSL:shutdown*** then behaves like
***SSL:set_shutdown*** called with SSL_SENT_SHUTDOWN|SSL_RECEIVED_SHUTDOWN.

The session is thus considered to be shutdown, but no "close notify" alert
is sent to the peer. This behaviour violates the TLS standard.

The default is normal shutdown behaviour as described by the TLS standard.

* ***ssl_ctx:verify_locations***(string CAfile[, string CAPath]) -> boolean
ssl_ctx:verify_locations specifies the locations for *ctx*, at
which CA certificates for verification purposes are located. The certificates
available via *CAfile* and *CApath* are trusted.

* ***ssl_ctx:cert_store***([openssl.x509_store store]) => [openssl.x509_store]
 * Given certstore sets/replaces the certificate verification storage of ***ctx*** to/with store.
 * If store is nil or none, another X509_STORE object is currently
set in ***ctx***, willl be return.

* ***ssl_ctx:verify_depth***([number depth]) -> number
 * Get verify depth when cert chain veirition, If given new depth, it will be used.

* ***ssl_ctx:verify_mode***() -> number, string
 * Return verify_mode number and string description

verify_mode list:
 
```
  none: not verify client cert
  peer: verify client cert
  fail: if client not have cert, will failure
  once: verify client only once.
```

* ***ssl_ctx:set_verify***(string mode=[none|peer][,function verifycb]) -> boolean
 * mode same with  ***ssl_ctx:mode***
 * verifycb should like int function(int preverify_ok,X509_STORE_CTX ctx)
   return 0 to end,1 to continue

* ***ssl_ctx:bio**(string host_addr[, boolean server=true[, boolean autoretry=true]]) => bio,ssl
 * Create bio and ssl object, if server is true,host_addr is listen bind address,
 
## SSL Object

* ***ssl:want***() -> string,number
 * Return want operation,should be: nothing,reading,writing,x509_lookup

* ***ssl:cipher***() -> table
 * Return current cipher info,table has key name,version,id,bits,description

* ***ssl:pending***() -> number
 * Return the number of bytes which are available inside ***SSL*** for immediate read. 

* ***ssl:ctx***([ssl_ctx ctx]) => ssl_ctx
 * Return ssl_ctx for SSL object, or set new SSL_CTX

* ***ssl:shutdown***()
 * Shutdown SSL connection

* ***ssl:shutdown***(string mode='read'|'write'|'quiet'|'nonoquiet')
 * Set shutdown mode, disable read or write, enable or disable quiet shutdown.
 
* ***ssl:shutdown***(boolean mode) -> string
 * If mode set true, return true for false for quiet,
 * Or return  'read' or 'write' for shutdown direction. 

* ***ssl:get***(string what,....) -> value,....
 * Return value according to what, args can be a list. arg must be in below list:

```
  certificate:  return SSL certificates
  fd: return file or network connect fd
  rfd:
  wfd:
  client_CA_list
  read_ahead: -> boolean
  shared_ciphers: string
  cipher_list -> string
  verify_mode: number
  verify_depth
  state_string
  state_string_long
  rstate_string
  rstate_string_long
  iversion
  version
  default_timeout,
  certificates
  verify_result
  state
  state_string
  
```

* ***ssl:set***(string what,value val,....) -> value,....
 * Set what with value, args must be key value pair

```
  certificate:  return SSL certificates
  fd: return file or network connect fd
  rfd:
  wfd:
  client_CA:
  read_ahead
  cipher_list
  verify_depth
  purpose:
  trust:
  verify_result
  state
```

#9. Misc Functions and Objects

##Funcions

* ***openssl.hex*** (string bin[, boolean encode=true]) -> hex string
 * Returns a string thant encode or decode from string.

* ***openssl.list*** (string 'digests'|'ciphers'|'pkeys'|'comps') -> array
 * Returns a method name array.

* ***openssl.error***([boolean verbose=false]) -> number, errmsg[, verbose]
 * Return openssl error message, should be call fellow fail method.

* ***openssl.random***(number length[, boolean strong]) -> string
 * Return a random string with length

* ***openssl.object***(string oid, string name [, string alias]) -> boolean
 * Add an object, return true for success or nil

* ***openssl.object***(number nid|string name) => asn1_object
 * Return asn1_object if exist or nil

* ***openssl.random*** (number length [, boolean strong=false]) -> string, boolean
 * Returns a string of the length specified filled with random bytes

* ***openssl.error*** ([boolean verbose=false]) -> number, string[,string]
 * If an error is found, it will return an error number code, followed by description string or it will return nothing and clear the error state, so you can call it twice.
 * When set verbose true, 3rd value returned is verbose message

* ***openssl.engine*** (string id) => openssl.engine


##Objects

###openssl.bio

***openssl.bio*** is a help object, it is useful, but rarely use.

* ***openssl.bio*** ([string data]) => bio
 * same with ***bio.mem***, implicaion by metatable "__call"

* ***openssl.bio.mem*** ([string data]) => bio
 * Create a memory bio, if data given, it will be memory buffer data
 * It can be input or output object.

* ***openssl.bio.socket*** (number socket[,string flag='noclose']) => bio
 * Create a socket bio, default not do socket close

* ***openssl.bio.dgram*** (number socket[,string flag='noclose']) => bio
 * Create a udp socket bio, default not do socket close

* ***openssl.bio.fd*** (number socket[,string flag='noclose']) => bio
 * Create a socket or file bio, default not do socket close

* ***openssl.bio.file*** (string file [,string mode='r']) => bio
 * Create a file bio, if mode not given, the default is 'r'

* ***openssl.bio.connect***(string host_port[, boolean connect=true])
 * Create network bio with 'host:port' address, if connect set true, will connect immediately.
 
* ***openssl.bio.accept***(string host_port)
 * Create network bio with 'host:port' address

* ***openssl.bio.filter***(string 'base64'|'buffer') => bio
 * Create base64 or buffer bio, which can append to an io BIO object

* ***openssl.bio.filter***(string 'digest', digest md) => bio
 * Create digest bio, which can append to an io BIO object

* ***openssl.bio.filter***(string 'ssl', ssl s[, string closeflag='noclose']) => bio
 * Create digest bio

* ***openssl.bio.filter***(string 'cipher', string key, stirng iv, boolean encrypt=true) => bio
 * Create cipher bio, which can append to an io BIO object

* ***bio:read*** (number len) -> string

* ***bio:gets*** ([number len=256]) -> string

* ***bio:write*** (string data) -> number

* ***bio:puts*** (string data) -> number

* ***bio:get_mem***() -> string
 * only supports bio mem

* ***bio:push(bio append)*** => bio
 * return end of chain bio object, if want to free a bio chain, use ***bio:free_all***()

* ***bio:pop(bio b)***
 * remove bio b from chian 

* ***bio:free_all()***
 * free a object chian

* ***bio:close*** ()

* ***bio:type*** () -> string

* ***bio:reset*** ()

###openssl.asn1_string

* ***openssl.asn1.string_new***([string type='octet']) => asn1_string
 * Create asn1_string, default will be 'octet' string.
 * type must be in:	"bit", "octet","utf8","numeric","printable","t61","teletex",
   "videotex","ia5","graphics","iso64","visible","general","unversal","bmp"

* ***asn1_string:len***() or ***`#asn1_string`***
 * Return length of string

* ***asn1_string:data***() -> string
 * Return raw data of string

* ***asn1_string:data***(string s) -> boolean
 * Set raw data to asn1_string

* ***asn1_string:dup***() => asn1_string

* ***asn1_string:toutf8***() -> string

* ***asn1_string:type*** => string

* ***asn1_string:equals***(asn1_string another) or ***asn1_string a==asn1_string b*** -> boolean
 * Return true if equals or false

###openssl.bn
* ***openssl.bn*** come from [http://www.tecgraf.puc-rio.br/~lhf/ftp/lua/](http://www.tecgraf.puc-rio.br/~lhf/ftp/lua/),thanks.

###openssl.engine
 ***openssl.engine*** is a help object, it can change openssl default action.


#A.   Howto

### Howto 1: Build on Linux/Unix System.

Before building, please change the setting in the config file.
Works with Lua5.1 (should support Lua5.2 by updating config file).

	make
	make install
	make clean

### Howto 2: Build on Windows with MSVC.

Before building, please change the setting in the config.win file.
Works with Lua5.1 (should support Lua5.2 by updating the config.win file).

	nmake -f makefile.win
	nmake -f makefile.win install
	nmake -f makefile.win clean


### Howto 3: Build on Windows with mingw.

TODO

#B.  Example usage

### Example 1: short encrypt/decrypt

```lua
local evp_cipher = openssl.cipher.get('des')
m = 'abcdefghick'
key = m
cdata = evp_cipher:encrypt(m,key)
m1  = evp_cipher:decrypt(cdata,key)
assert(cdata==m1)
```

### Example 2: quick evp_digest

```lua
md = openssl.digest.get('md5')
m = 'abcd'
aa = md:evp_digest(m)

mdc=md:init()
mdc:update(m)
bb = mdc:final()
assert(aa==bb)
```

### Example 3:  Iterate a openssl.stack_of_x509(sk_x509) object

```lua
n = #sk
for i=1, n do
	x = sk:get(i)
end
```

### Example 4: read and parse certificate

```lua
local openssl = require('openssl')

function dump(t,i)
	for k,v in pairs(t) do
		if(type(v)=='table') then
			print( string.rep('\t',i),k..'={')
			dump(v,i+1)
			print( string.rep('\t',i),k..'=}')
		else
			print( string.rep('\t',i),k..'='..tostring(v))
		end
	end
end

function test_x509()
	local x = openssl.x509.read(certasstring)
	print(x)
	t = x:parse()
	dump(t,0)
	print(t)
end

test_x509()
```

###Example 5: bio network handle(TCP)

 * server
 
```lua
local openssl = require'openssl'
local bio = openssl.bio

host = host or "127.0.0.1"; --only ip
port = port or "8383";

local srv = assert(bio.accept(host..':'..port))
print('listen at:'..port)
local cli = assert(srv:accept())
while 1 do
    cli = assert(srv:accept())
    print('CLI:',cli)
    while cli do
        local s = assert(cli:read())
        print(s)
        assert(cli:write(s))
    end
    print(openssl.error(true))
end
```

 * client
```lua
local openssl = require'openssl'
local bio = openssl.bio
io.read()

host = host or "127.0.0.1"; --only ip
port = port or "8383";

local cli = assert(bio.connect(host..':'..port,true))

    while cli do
        s = io.read()
        if(#s>0) then
            print(cli:write(s))
            ss = cli:read()
            assert(#s==#ss)
        end
    end
    print(openssl.error(true))
```

For more examples, please see test lua script file.

#C. Contact


--------------------------------------------------------------------
***lua-openssl License***

Copyright (c) 2011 - 2014 zhaozg, zhaozg(at)gmail.com

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to
deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

--------------------------------------------------------------------

This product includes PHP software, freely available from <http://www.php.net/software/>

