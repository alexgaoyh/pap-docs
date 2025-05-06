# [SSL] Tomcat10下的多个SSL证书配置

## 介绍

&ensp;&ensp;近期协助朋友做一个技术方案（CMS中多级子站的实现），为了减少后续可能的维护成本，需要尽可能少的使用组件，所以想要将SSL证书的部分从Nginx中迁移到Tomcat，从而不再使用Nginx。

&ensp;&ensp;基于如上尽可能少的使用组件的背景，特写此文。

## 使用方法

1. 测试证书生成，本地使用 openssl 生成两套测试的证书：

    ```shell
        # 生成私钥
        openssl genpkey -algorithm RSA -out privkey1.pem -aes256 -pass pass:alexgaoyh
        
        # 生成证书签名请求 (CSR)
        openssl req -new -key privkey1.pem -out csr1.pem -passin pass:alexgaoyh
        
        # 生成自签名证书
        openssl x509 -req -in csr1.pem -signkey privkey1.pem -out fullchain1.pem -days 1
        
        # 将私钥和证书转换为 .p12 格式
        openssl pkcs12 -export -in fullchain1.pem -inkey privkey1.pem -out example1.p12 -name example1 -password pass:alexgaoyh
        
        =====================================================================================================================================================
  
        openssl genpkey -algorithm RSA -out privkey2.pem -aes256 -pass pass:alexgaoyh
        openssl req -new -key privkey2.pem -out csr2.pem -passin pass:alexgaoyh
        openssl x509 -req -in csr2.pem -signkey privkey2.pem -out fullchain2.pem -days 1
        openssl pkcs12 -export -in fullchain2.pem -inkey privkey2.pem -out example2.p12 -name example1 -password pass:alexgaoyh
    ```
   
2. 云服务器证书
    1. 在云厂商中生成证书，并且使用可用于Nginx中的证书： *.pem *.key 两个文件

3. 其他
   1. OpenSSL configuration file
   
   ```properties
    [req]
    distinguished_name = req_distinguished_name
    x509_extensions = v3_ca
    prompt = no
    default_bits = 2048
    default_keyfile = privkey.pem
    default_md = sha256
    
    [req_distinguished_name]
    C = US
    ST = State
    L = City
    O = Organization
    OU = Organizational Unit
    CN = example2.pap.net.cn
    
    [v3_ca]
    basicConstraints = critical,CA:TRUE
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer:always
    ```
   2. 查看证书信息
   
   ```shell
    keytool -list -v -keystore example1.p12 -storetype PKCS12
    ```
   
   3. application.yml 配置
   
   ```properties
   server:
    ssl:
        key-store: classpath:ssl/example1.p12
        key-store-password: alexgaoyh
        key-store-type: PKCS12
        key-alias: example1
   ```


## 基于 Spring Boot3 的 Tomcat10 的配置文件

1. 基于Spring Boot3 的，默认使用Tomcat10，并且如下代码仅支持 Tomcat10 .
    ```java
        package cn.net.pap.example.admin.config.ssl;

        import org.apache.tomcat.util.net.SSLHostConfig;
        import org.apache.tomcat.util.net.SSLHostConfigCertificate;
        import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
        import org.springframework.boot.web.server.WebServerFactoryCustomizer;
        import org.springframework.context.annotation.Configuration;
        import org.springframework.core.io.DefaultResourceLoader;
        import org.springframework.core.io.Resource;
        
        import java.io.IOException;
        import java.io.UncheckedIOException;
        import java.util.Map;
        
        @Configuration
        public class MultiSslTomcatCustomizer implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {
        
            private final Map<String, SslConfig> sslConfigs = Map.of(
                    "example1.pap.net.cn", new Pkcs12SslConfig("classpath:ssl/example1.p12", "alexgaoyh"),
                    "example2.pap.net.cn", new Pkcs12SslConfig("classpath:ssl/example2.p12", "alexgaoyh"),
                    "pap-docs.pap.net.cn", new PemSslConfig("classpath:ssl/pap-docs.pap.net.cn_bundle.pem", "classpath:ssl/pap-docs.pap.net.cn.key")
            );
        
            @Override
            public void customize(TomcatServletWebServerFactory factory) {
                factory.addConnectorCustomizers(connector -> {
                    connector.setPort(443);
                    connector.setSecure(true);
                    connector.setScheme("https");
        
                    for (Map.Entry<String, SslConfig> entry : sslConfigs.entrySet()) {
                        String domain = entry.getKey();
                        SslConfig ssl = entry.getValue();
        
                        SSLHostConfig hostConfig = new SSLHostConfig();
                        hostConfig.setHostName(domain);
                        hostConfig.setSslProtocol("TLS");
        
                        SSLHostConfigCertificate cert = new SSLHostConfigCertificate(hostConfig, SSLHostConfigCertificate.Type.RSA);
        
                        if (ssl instanceof Pkcs12SslConfig pkcs12) {
                            cert.setCertificateKeystoreFile(resolvePath(pkcs12.keystore));
                            cert.setCertificateKeystorePassword(pkcs12.password);
                            cert.setCertificateKeystoreType("PKCS12");
                        } else if (ssl instanceof PemSslConfig pem) {
                            cert.setCertificateFile(resolvePath(pem.certFile));
                            cert.setCertificateKeyFile(resolvePath(pem.keyFile));
                        } else {
                            throw new IllegalArgumentException("Unsupported SSL config type: " + ssl.getClass());
                        }
        
                        hostConfig.addCertificate(cert);
                        connector.addSslHostConfig(hostConfig);
                    }
                });
            }
        
            private String resolvePath(String location) {
                try {
                    Resource resource = new DefaultResourceLoader().getResource(location);
                    return resource.getFile().getAbsolutePath();
                } catch (IOException e) {
                    throw new UncheckedIOException("Cannot resolve path: " + location, e);
                }
            }
        
            private sealed interface SslConfig permits Pkcs12SslConfig, PemSslConfig {}
        
            private static final class Pkcs12SslConfig implements SslConfig {
                final String keystore;
                final String password;
        
                Pkcs12SslConfig(String keystore, String password) {
                    this.keystore = keystore;
                    this.password = password;
                }
            }
        
            private static final class PemSslConfig implements SslConfig {
                final String certFile;
                final String keyFile;
        
                PemSslConfig(String certFile, String keyFile) {
                    this.certFile = certFile;
                    this.keyFile = keyFile;
                }
            }
        }    
    ```

## 相关参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
