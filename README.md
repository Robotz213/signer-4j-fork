# Signer4j

> **Disclaimer:** Este projeto é um fork do repositório original de [l3onardo-oliv3ira](https://github.com/l3onardo-oliv3ira/signer4j).

Uma forma mais simples de realizar operações de assinatura digital com certificados A1 e A3.

## Instalação

### Dependências

- [Progress4j](https://github.com/l3onardo-oliv3ira/progress4j)
- [Utils4j](https://github.com/l3onardo-oliv3ira/utils4j)

#### Adicionando dependências

Para adicionar as dependências ao seu projeto, inclua as seguintes linhas no seu arquivo `pom.xml`:

```xml
<dependency>
  <groupId>com.github.l3onardo-oliv3ira</groupId>
  <artifactId>progress4j</artifactId>
  <version>2.6.0-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.github.l3onardo-oliv3ira</groupId>
  <artifactId>utils4j</artifactId>
  <version>2.6.0-SNAPSHOT</version>
</dependency>
```

E instalar com o comando

```bash
mvn install:install-file -Dfile=path/to/progress4j-2.6.0-SNAPSHOT.jar -DgroupId=com.github.l3onardo-oliv3ira -DartifactId=progress4j -Dversion=2.6.0-SNAPSHOT -Dpackaging=jar

mvn install:install-file -Dfile=path/to/utils4j-2.6.0-SNAPSHOT.jar -DgroupId=com.github.l3onardo-oliv3ira -DartifactId=utils4j -Dversion=2.6.0-SNAPSHOT -Dpackaging=jar

```

## Listando dispositivos detectados automaticamente

```java
IDeviceManager dm = new DeviceManager();

dm.getDevices().stream().forEach(d -> {
  IDevice device = d;
  System.out.println("Driver: " + device.getDriver());
  System.out.println("Etiqueta: " + device.getLabel());
  System.out.println("Modelo: " + device.getModel());
  System.out.println("Serial: " + device.getSerial());
});
```

#### Interface do gerenciador de dispositivos

```java
public interface IDeviceManager {
  List<IDevice> getDevices();
  List<IDevice> getDevices(boolean forceReload);
  List<IDevice> getDevices(Predicate<IDevice> predicate);
  List<IDevice> getDevices(Predicate<IDevice> predicate, boolean forceReload);

  Optional<IDevice> firstDevice();
  Optional<IDevice> firstDevice(boolean forceReload);
  Optional<IDevice> firstDevice(Predicate<IDevice> predicate);
  Optional<IDevice> firstDevice(Predicate<IDevice> predicate, boolean forceReload);

  void close();
}
```

#### Interface do dispositivo

```java
public interface ISerialItem {
  String getManufacturer();
  String getSerial();
}
public interface IGadget extends ISerialItem {
  String getLabel();
  String getModel();
}
public interface IDevice extends IGadget {
  enum Type { A1, A3 }
  Type getType();
  String getDriver();
  ISlot getSlot();
  ICertificates getCertificates();
}
```

## Acessando o primeiro dispositivo e listando certificados

```java
IDevice device = dm.firstDevice().get();
System.out.println("Certificados");
device.getCertificates().stream().forEach(c -> {
  System.out.println("---------");
  System.out.println("Nome: " + c.getName());
  System.out.println("Data de início: " + c.getAfterDate());
  System.out.println("Data de expiração: " + c.getBeforeDate());
  System.out.println("Email: " + c.getEmail().orElse("Desconhecido"));
  //.... c.get*

  if (certificate.hasCertificatePF()) {
   ICertificatePF pf = c.getCertificatePF().get();
   System.out.println("CPF: " + pf.getCPF().get());
   //.... pf.get*

  }
  if (certificate.hasCertificatePJ()) {
   ICertificatePJ pj = c.getCertificatePJ().get();
   System.out.println("CNPJ: " + pj.getCNPJ().get());
   //.... pj.get*

  }
});
```

#### Interface do certificado

```java
public interface ICertificate extends ISerialItem {
  Date getAfterDate();
  Date getBeforeDate();

  IDistinguishedName getCertificateIssuerDN();
  IDistinguishedName getCertificateSubjectDN();

  List<String> getCRLDistributionPoint() throws IOException;

  Optional<String> getEmail();
  Optional<ICertificatePF> getCertificatePF(); // abstração completa para Pessoa Física
  Optional<ICertificatePJ> getCertificatePJ(); // abstração completa para Pessoa Jurídica

  KeyUsage getKeyUsage();
  ISubjectAlternativeNames getSubjectAlternativeNames();

  String getName();

  X509Certificate toX509();

  boolean hasCertificatePF();
  boolean hasCertificatePJ();
}
```

## Abstração brasileira para certificados de pessoa física e jurídica

```java
public interface ICertificatePF {
  Optional<String> getCPF();
  Optional<LocalDate> getBirthDate();
  Optional<String> getNis();
  Optional<String> getRg();
  Optional<String> getIssuingAgencyRg();
  Optional<String> getUfIssuingAgencyRg();
  Optional<String> getElectoralDocument();
  Optional<String> getSectionElectoralDocument();
  Optional<String> getZoneElectoralDocument();
  Optional<String> getCityElectoralDocument();
  Optional<String> getUFElectoralDocument();
  Optional<String> getCEI();
}

public interface ICertificatePJ {
  Optional<String> getResponsibleName();
  Optional<String> getResponsibleCPF();
  Optional<String> getCNPJ();
  Optional<String> getBirthDate();
  Optional<String> getBusinessName();
  Optional<String> getNis();
  Optional<String> getRg();
  Optional<String> getIssuingAgencyRg();
  Optional<String> getUfIssuingAgencyRg();
  Optional<String> getCEI();
}
```

## Exibindo informações do slot

```java
ISlot slot = device.getSlot();

System.out.println("Informações do slot");
System.out.println("Descrição: " + slot.getDescription());
System.out.println("Versão do firmware: " + slot.getFirmewareVersion());
System.out.println("Versão do hardware: " + slot.getHardwareVersion());
System.out.println("Fabricante: " + slot.getManufacturer());
System.out.println("Número: " + slot.getNumber());
System.out.println("Serial: " + slot.getSerial());
```

#### Interface do slot

```java
public interface ISlot extends ISerialItem {
  long getNumber();
  IToken getToken();
  IDevice toDevice();
  String getDescription();
  String getHardwareVersion();
  String getFirmewareVersion();
}
```

## Abstração de token para um slot

```java
IToken token = slot.getToken();

try {
  token.login();

  String mensagem = "Olá mundo!";

  ISimpleSigner simpleSigner = token.signerBuilder().build();

  ISignedData data = simpleSigner.process(mensagem);

  System.out.println("dados assinados base64: " + data.getSignature64());
} catch (TokenLockedException e) {
  System.out.println("Seu token está bloqueado");
} catch (InvalidPinException e) {
  System.out.println("Sua senha está incorreta!");
} catch (NoTokenPresentException e) {
  System.out.println("Seu token não está conectado ao USB");
} catch (LoginCanceledException e) {
  System.out.println("Autenticação cancelada pelo usuário");
} catch (Signer4JException e) {
  System.out.println(e.getMessage());
} finally {
  token.logout();
}
```

#### Interface de dados assinados

```java
public interface IPersonalData {
  PrivateKey getPrivateKey();
  Certificate getCertificate();
  List<Certificate> getCertificateChain();
  String getCertificate64() throws CertificateException;
  String getCertificateChain64() throws CertificateException;
  int chainSize();
}

public interface ISignedData extends IPersonalData {
  byte[] getSignature();
  String getSignature64();
  void writeTo(OutputStream out) throws IOException;
}
```

#### Interface do token

```java
public interface IToken extends IGadget {
  IToken login() throws Signer4JException;
  IToken login(char[] password) throws Signer4JException;
  IToken login(IPasswordCollector collector) throws Signer4JException;
  IToken login(IPasswordCallbackHandler callback) throws Signer4JException;

  void logout();

  long getMinPinLen();
  long getMaxPinLen();

  String getManufacturer();
  boolean isAuthenticated();

  ISlot getSlot();
  TokenType getType();

  ICertificates getCertificates();
  IKeyStoreAccess getKeyStoreAccess();

  ISignerBuilder signerBuilder();
  ISignerBuilder signerBuilder(ICertificateChooserFactory factory);

  ICMSSignerBuilder cmsSignerBuilder();
  ICMSSignerBuilder cmsSignerBuilder(ICertificateChooserFactory factory);

  IPKCS7SignerBuilder pkcs7SignerBuilder();
  IPKCS7SignerBuilder pkcs7SignerBuilder(ICertificateChooserFactory factory);
}
```

## Assinando arquivo para saída .p7s

```java
try {
  token.login();

  ICMSSigner cmsSigner = token.cmsSignerBuilder()
    .usingAlgorithm(SignatureAlgorithm.SHA1withRSA)
    .usingAttributes(true)
    .usingMemoryLimit(50 * 1024 * 1024)
    .usingSignatureType(SignatureType.ATTACHED)
    .build();

  ISignedData data = cmsSigner.process(new File("./input.pdf"));

  try(OutputStream out = new FileOutputStream(new File("./input.pdf.p7s"))) {
   data.writeTo(out);
  }
} catch (TokenLockedException e) {
  System.out.println("Seu token está bloqueado");
} catch (InvalidPinException e) {
  System.out.println("Sua senha está incorreta!");
} catch (NoTokenPresentException e) {
  System.out.println("Seu token não está conectado ao USB");
} catch (LoginCanceledException e) {
  System.out.println("Autenticação cancelada pelo usuário");
} catch (Signer4JException e) {
  System.out.println(e.getMessage());
} catch (IOException e) {
  System.out.println("Não foi possível ler o arquivo de entrada");
} finally {
  token.logout();
}
```

#### Interface de processamento de bytes

```java
public interface IByteProcessor {
  ISignedData process(byte[] content, int offset, int length) throws Signer4JException;
  ISignedData process(byte[] content) throws Signer4JException;
  ISignedData process(File content) throws Signer4JException, IOException;
  ISignedData process(String content) throws Signer4JException;
  ISignedData process(String content, Charset charset) throws Signer4JException;
  byte[] processRaw(byte[] content) throws Signer4JException;
  byte[] processRaw(String content) throws Signer4JException;
  byte[] processRaw(String content, Charset charset) throws Signer4JException;
  String process64(byte[] content) throws Signer4JException;
  String process64(String content) throws Signer4JException;
  String process64(String content, Charset charset) throws Signer4JException;
  String process64(File input) throws Signer4JException, IOException;
}
```
