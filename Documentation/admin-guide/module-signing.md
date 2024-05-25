---
tip: translate by baidu@2024-01-30 21:36:08
---
---
title: Kernel module signing facility
---

# Overview


The kernel module signing facility cryptographically signs modules during installation and then checks the signature upon loading the module. This allows increased kernel security by disallowing the loading of unsigned modules or modules signed with an invalid key. Module signing increases security by making it harder to load a malicious module into the kernel. The module signature checking is done by the kernel so that it is not necessary to have trusted userspace bits.

> 内核模块签名功能在安装过程中对模块进行加密签名，然后在加载模块时检查签名。这允许通过禁止加载未签名的模块或使用无效密钥签名的模块来提高内核安全性。模块签名使恶意模块更难加载到内核中，从而提高了安全性。模块签名检查是由内核完成的，因此不需要有可信的用户空间位。


This facility uses X.509 ITU-T standard certificates to encode the public keys involved. The signatures are not themselves encoded in any industrial standard type. The built-in facility currently only supports the RSA & NIST P-384 ECDSA public key signing standard (though it is pluggable and permits others to be used). The possible hash algorithms that can be used are SHA-2 and SHA-3 of sizes 256, 384, and 512 (the algorithm is selected by data in the signature).

> 该设施使用X.509 ITU-T标准证书对所涉及的公钥进行编码。签名本身没有以任何工业标准类型进行编码。该内置设施目前仅支持RSA和NIST P-384 ECDSA公钥签名标准（尽管它是可插拔的，并允许使用其他标准）。可以使用的可能的哈希算法是大小为256、384和512的SHA-2和SHA-3（该算法由签名中的数据选择）。

# Configuring module signing


The module signing facility is enabled by going to the `Enable Loadable Module Support`{.interpreted-text role="menuselection"} section of the kernel configuration and turning on:

> 模块签名功能可通过转到内核配置的“启用可加载模块支持”｛.depreted text role=“menuselection”｝部分并打开：

    CONFIG_MODULE_SIG   "Module signature verification"

This has a number of options available:

> (1) `Require modules to be validly signed`{.interpreted-text role="menuselection"} (`CONFIG_MODULE_SIG_FORCE`)
>
>     This specifies how the kernel should deal with a module that has a signature for which the key is not known or a module that is unsigned.
>
>     If this is off (ie. \"permissive\"), then modules for which the key is not available and modules that are unsigned are permitted, but the kernel will be marked as being tainted, and the concerned modules will be marked as tainted, shown with the character \'E\'.
>
>     If this is on (ie. \"restrictive\"), only modules that have a valid signature that can be verified by a public key in the kernel\'s possession will be loaded. All other modules will generate an error.
>
>     Irrespective of the setting here, if the module has a signature block that cannot be parsed, it will be rejected out of hand.
>
> (2) `Automatically sign all modules`{.interpreted-text role="menuselection"} (`CONFIG_MODULE_SIG_ALL`)
>
>     If this is on then modules will be automatically signed during the modules_install phase of a build. If this is off, then the modules must be signed manually using:
>
> > scripts/sign-file
>
> \(3\) `Which hash algorithm should modules be signed with?`{.interpreted-text role="menuselection"}
>
> > This presents a choice of which hash algorithm the installation phase will sign the modules with:
> >
> > > =============================== ==========================================
> >
> > `CONFIG_MODULE_SIG_SHA256` `Sign modules with SHA-256`{.interpreted-text role="menuselection"} `CONFIG_MODULE_SIG_SHA384` `Sign modules with SHA-384`{.interpreted-text role="menuselection"} `CONFIG_MODULE_SIG_SHA512` `Sign modules with SHA-512`{.interpreted-text role="menuselection"} `CONFIG_MODULE_SIG_SHA3_256` `Sign modules with SHA3-256`{.interpreted-text role="menuselection"} `CONFIG_MODULE_SIG_SHA3_384` `Sign modules with SHA3-384`{.interpreted-text role="menuselection"} `CONFIG_MODULE_SIG_SHA3_512` `Sign modules with SHA3-512`{.interpreted-text role="menuselection"} =============================== ==========================================
> >
> > The algorithm selected here will also be built into the kernel (rather than being a module) so that modules signed with that algorithm can have their signatures checked without causing a dependency loop.
>
> (4) `File name or PKCS#11 URI of module signing key`{.interpreted-text role="menuselection"} (`CONFIG_MODULE_SIG_KEY`)
>
>     Setting this option to something other than its default of `certs/signing_key.pem` will disable the autogeneration of signing keys and allow the kernel modules to be signed with a key of your choosing. The string provided should identify a file containing both a private key and its corresponding X.509 certificate in PEM form, or --- on systems where the OpenSSL ENGINE_pkcs11 is functional --- a PKCS#11 URI as defined by RFC7512. In the latter case, the PKCS#11 URI should reference both a certificate and a private key.
>
>     If the PEM file containing the private key is encrypted, or if the PKCS#11 token requires a PIN, this can be provided at build time by means of the `KBUILD_SIGN_PIN` variable.
>
> (5) `Additional X.509 keys for default system keyring`{.interpreted-text role="menuselection"} (`CONFIG_SYSTEM_TRUSTED_KEYS`)
>
>     This option can be set to the filename of a PEM-encoded file containing additional certificates which will be included in the system keyring by default.


Note that enabling module signing adds a dependency on the OpenSSL devel packages to the kernel build processes for the tool that does the signing.

> 请注意，启用模块签名会将对OpenSSL-devel包的依赖添加到执行签名的工具的内核构建过程中。

# Generating signing keys


Cryptographic keypairs are required to generate and check signatures. A private key is used to generate a signature and the corresponding public key is used to check it. The private key is only needed during the build, after which it can be deleted or stored securely. The public key gets built into the kernel so that it can be used to check the signatures as the modules are loaded.

> 生成和检查签名需要加密密钥对。私钥用于生成签名，并使用相应的公钥进行检查。私钥只在构建过程中需要，之后可以安全地删除或存储。公钥被构建到内核中，以便在加载模块时可以使用它来检查签名。


Under normal conditions, when `CONFIG_MODULE_SIG_KEY` is unchanged from its default, the kernel build will automatically generate a new keypair using openssl if one does not exist in the file:

> 在正常情况下，当“CONFIG_MODULE_SIG_KEY”与默认值保持不变时，内核构建将使用openssl自动生成一个新的密钥对（如果文件中不存在）：

    certs/signing_key.pem


during the building of vmlinux (the public part of the key needs to be built into vmlinux) using parameters in the:

> 在构建vmlinux期间（密钥的公共部分需要构建到vmlinux中），使用中的参数：

    certs/x509.genkey

file (which is also generated if it does not already exist).


One can select between RSA (`MODULE_SIG_KEY_TYPE_RSA`) and ECDSA (`MODULE_SIG_KEY_TYPE_ECDSA`) to generate either RSA 4k or NIST P-384 keypair.

> 可以在RSA（`MODULE_SIG_KEY_TYPE_RSA`）和ECDSA（`MODURE_SIG_KEY_TYPE_ECDSA`）之间进行选择，以生成RSA 4k或NIST P-384密钥对。

It is strongly recommended that you provide your own x509.genkey file.


Most notably, in the x509.genkey file, the req_distinguished_name section should be altered from the default:

> 最值得注意的是，在x509.genkey文件中，req_distinguished_name部分应该从默认值更改为：

    [ req_distinguished_name ]
    #O = Unspecified company
    CN = Build time autogenerated kernel key
    #emailAddress = unspecified.user@unspecified.company

The generated RSA key size can also be set with:

    [ req ]
    default_bits = 4096


It is also possible to manually generate the key private/public files using the x509.genkey key generation configuration file in the root node of the Linux kernel sources tree and the openssl command. The following is an example to generate the public/private key files:

> 也可以使用Linux内核源代码树的根节点中的x509.genkey密钥生成配置文件和openssl命令手动生成密钥专用/公共文件。以下是生成公钥/私钥文件的示例：

    openssl req -new -nodes -utf8 -sha256 -days 36500 -batch -x509 \
       -config x509.genkey -outform PEM -out kernel_key.pem \
       -keyout kernel_key.pem


The full pathname for the resulting kernel_key.pem file can then be specified in the `CONFIG_MODULE_SIG_KEY` option, and the certificate and key therein will be used instead of an autogenerated keypair.

> 然后，可以在“CONFIG_MODULE_SIG_key”选项中指定生成的kernel_key.pem文件的完整路径名，并且其中的证书和密钥将被使用，而不是自动生成的密钥对。

# Public keys in the kernel


The kernel contains a ring of public keys that can be viewed by root. They\'re in a keyring called \".builtin_trusted_keys\" that can be seen by:

> 内核包含一个可以由root查看的公钥环。它们位于一个名为“.buildin_trusted_keys”的钥匙圈中，可以通过以下方式查看：

    [root@deneb ~]# cat /proc/keys
    ...
    223c7853 I------     1 perm 1f030000     0     0 keyring   .builtin_trusted_keys: 1
    302d2d52 I------     1 perm 1f010000     0     0 asymmetri Fedora kernel signing key: d69a84e6bce3d216b979e9505b3e3ef9a7118079: X509.RSA a7118079 []
    ...


Beyond the public key generated specifically for module signing, additional trusted certificates can be provided in a PEM-encoded file referenced by the `CONFIG_SYSTEM_TRUSTED_KEYS` configuration option.

> 除了专门为模块签名生成的公钥之外，还可以在“CONFIG_SYSTEM_trusted_KEYS”配置选项引用的PEM编码文件中提供额外的可信证书。


Further, the architecture code may take public keys from a hardware store and add those in also (e.g. from the UEFI key database).

> 此外，体系结构代码可以从硬件存储中获取公钥，并将这些公钥也添加到（例如，从UEFI密钥数据库）中。

Finally, it is possible to add additional public keys by doing:

    keyctl padd asymmetric "" [.builtin_trusted_keys-ID] <[key-file]

e.g.:

    keyctl padd asymmetric "" 0x223c7853 <my_public_key.x509


Note, however, that the kernel will only permit keys to be added to `.builtin_trusted_keys` **if** the new key\'s X.509 wrapper is validly signed by a key that is already resident in the `.builtin_trusted_keys` at the time the key was added.

> 但是，请注意，只有当**新密钥的X.509包装由添加密钥时已驻留在“.building_trusted_keys”中的密钥有效签名时，内核才会允许将密钥添加到“.buildin-trusted_keys”**中。

# Manually signing modules


To manually sign a module, use the scripts/sign-file tool available in the Linux kernel source tree. The script requires 4 arguments:

> 要手动对模块进行签名，请使用Linux内核源代码树中提供的脚本/签名文件工具。脚本需要4个参数：

> 1.  The hash algorithm (e.g., sha256)
> 2.  The private key filename or PKCS#11 URI
> 3.  The public key filename
> 4.  The kernel module to be signed

The following is an example to sign a kernel module:

    scripts/sign-file sha512 kernel-signkey.priv \
        kernel-signkey.x509 module.ko


The hash algorithm used does not have to match the one configured, but if it doesn\'t, you should make sure that hash algorithm is either built into the kernel or can be loaded without requiring itself.

> 所使用的哈希算法不必与配置的哈希算法匹配，但如果不匹配，则应确保哈希算法内置在内核中，或者可以在不需要自身的情况下加载。


If the private key requires a passphrase or PIN, it can be provided in the \$KBUILD_SIGN_PIN environment variable.

> 如果私钥需要密码短语或PIN，则可以在\$KBUILD_SIGN_PIN环境变量中提供。

# Signed modules and stripping


A signed module has a digital signature simply appended at the end. The string `~Module signature appended~.` at the end of the module\'s file confirms that a signature is present but it does not confirm that the signature is valid!

> 一个已签名的模块在末尾简单地附加了一个数字签名。字符串`~附加的模块签名~.`在模块文件的末尾，确认签名存在，但不能确认签名有效！


Signed modules are BRITTLE as the signature is outside of the defined ELF container. Thus they MAY NOT be stripped once the signature is computed and attached. Note the entire module is the signed payload, including any and all debug information present at the time of signing.

> 签名模块是BRITTLE，因为签名在定义的ELF容器之外。因此，一旦计算并附上签名，它们就不会被剥离。请注意，整个模块是已签名的有效负载，包括签名时存在的任何和所有调试信息。

# Loading signed modules


Modules are loaded with insmod, modprobe, `init_module()` or `finit_module()`, exactly as for unsigned modules as no processing is done in userspace. The signature checking is all done within the kernel.

> 模块加载有insmod、modprobe、`init_module（）`或`fini_module（）`，与未签名模块完全相同，因为在用户空间中不进行任何处理。签名检查都是在内核中完成的。

# Non-valid signatures and unsigned modules


If `CONFIG_MODULE_SIG_FORCE` is enabled or module.sig_enforce=1 is supplied on the kernel command line, the kernel will only load validly signed modules for which it has a public key. Otherwise, it will also load modules that are unsigned. Any module for which the kernel has a key, but which proves to have a signature mismatch will not be permitted to load.

> 如果启用了“CONFIG_MODULE_SIG_FORCE”或在内核命令行上提供了MODULE.SIG_enforce=1，则内核将仅加载具有公钥的有效签名模块。否则，它还会加载未签名的模块。内核有密钥但被证明签名不匹配的任何模块都不允许加载。

Any module that has an unparsable signature will be rejected.

# Administering/protecting the private key


Since the private key is used to sign modules, viruses and malware could use the private key to sign modules and compromise the operating system. The private key must be either destroyed or moved to a secure location and not kept in the root node of the kernel source tree.

> 由于私钥用于对模块进行签名，因此病毒和恶意软件可能会使用私钥对模块进行签署并危害操作系统。私钥必须被销毁或移动到安全位置，而不是保存在内核源树的根节点中。


If you use the same private key to sign modules for multiple kernel configurations, you must ensure that the module version information is sufficient to prevent loading a module into a different kernel. Either set `CONFIG_MODVERSIONS=y` or ensure that each configuration has a different kernel release string by changing `EXTRAVERSION` or `CONFIG_LOCALVERSION`.

> 如果使用相同的私钥为多个内核配置的模块签名，则必须确保模块版本信息足以防止将模块加载到不同的内核中。设置“CONFIG_MODVERSIONS=y”，或者通过更改“EXTRAVERSION”或“CONFIG_LOCALVERSION”确保每个配置都有不同的内核发布字符串。
