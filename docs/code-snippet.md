##JAVA Sample Code
Merchants can refer to these code snippets for onboarding the login flows.
The following code snippets can be copy/pasted directly and are ready for use.

###Helper Classes

####UltraConstants.java

This class contains constants that are used throughout the code.

```
public class UltraConstants {

    public static class ULTRA_SERVICE_PATHS {
        public static final String PLATFORM_PATH = "https://platform.flipkart.net";
        public static final String JWT_VALIDATION_PATH = "/1/dummy";
        public static final String ACCESS_TOKEN_PATH = "/1/authorization/auth";
        public static final String BULK_RESOURCE_PATH = "/2/resource/bulk";
    }

    public static class HEADERS {
        public static final String SECURE_TOKEN = "secureToken";
        public static final String CONTENT_TYPE = "Content-Type";
        public static final String JSON = "application/json";
        public static final String CACHE_CONTROL = "Cache-Control";
        public static final String NO_CACHE = "no-cache";
    }

    public static class QUERY_PARAMETERS {
        public static final String GRANT_TOKEN = "grantToken";
        public static final String CLIENT_ID = "clientId";
        public static final String CLIENT_SECRET = "clientSecret";
        public static final String ACCESS_TOKEN = "accessToken";
    }

    public static class JWT_TOKEN {
        public static final String ALG = "alg";
        public static final String RS256 = "rs256";
        public static final String TYP = "typ";
        public static final String JWT = "jwt";
        public static final String RSA = "RSA";
        public static final String EXPIRY_TIME = "1000000";
    }

}
```
####Response.java

 This is a generic class to store all the response types.

```

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data   
public class Response<T> {

    @JsonProperty("STATUS")
    String status;

    @JsonProperty("RESPONSE")
    T response;
}
```

####AccessTokenResponse.java

This class is used to store Access Token Response.

```

import lombok.Data;

@Data   
public class AccessTokenResponse { 
 
    private String identityToken;   
    private String accessToken;     
}
```

##Dependencies

The following Maven dependies are required to be imported in the code :
```
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.6.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle.bcprov-jdk15on.1.57.org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.57</version>
        </dependency>
        <dependency>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-client</artifactId>
            <version>1.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.4</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.5</version>
        </dependency>

```
###Generate Private and Public Key Pair

On the command line enter the following command to generate private and public key pair :

ssh-keygen -t rsa -b 4096 -m PEM -f private.key

openssl rsa -in private.key -pubout -outform PEM -out public.key.pub

private.key is the private key which is used to generate JWT token and should be kept secret by the partners.

public.key.pub is the public key that should be shared with Flipkart inorder to validate the partner.

###Generate JWT Token
This class is used to generate JWT token which is used to enable
 secure server to server communication between partner and Ultra.

```
import java.io.DataInputStream;     
import java.io.File;    
import java.io.FileInputStream;     
import java.io.IOException;     
import java.security.KeyFactory;        
import java.security.NoSuchAlgorithmException;      
import java.security.PrivateKey;        
import java.security.spec.InvalidKeySpecException;      
import java.security.spec.PKCS8EncodedKeySpec;      
import java.util.Base64;        
import java.util.Date;      
import java.util.HashMap;       
import java.util.Map;    
import <UltraConstants.java class>   

import io.jsonwebtoken.JwtBuilder;      
import io.jsonwebtoken.Jwts;        
import io.jsonwebtoken.SignatureAlgorithm;

public class GenerateJWTToken {

    private String privateKey;

    /**
     * @param privateKey
     */
    
    public GenerateJWTToken(String privateKey) {
        this.privateKey = privateKey;
    }

    /**
     * @param privKey
     * @throws IOException
     */
    public GenerateJWTToken(File privKey) throws IOException {

        // Loading the private key from the file
        FileInputStream fis = new FileInputStream(privKey);
        DataInputStream dis = new DataInputStream(fis);
        byte[] keyBytes = new byte[(int) privKey.length()];
        dis.readFully(keyBytes);
        dis.close();
        this.privateKey = new String(keyBytes);
    }

    /**
     * @param issuer the issuer of the JWT token
     * @return JWT token
     */
    public String encode(String issuer) throws InvalidKeySpecException, NoSuchAlgorithmException {

        String retStr = null;

        //Specifies the encoding algorithm to be used
        final SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.RS256;

        // Assigns the issue time of the JWT token
        Long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);

        //Describes the headers
        Map<String, Object> headers = new HashMap<>();
        headers.put(UltraConstants.JWT_TOKEN.ALG, UltraConstants.JWT_TOKEN.RS256);
        headers.put(UltraConstants.JWT_TOKEN.TYP, UltraConstants.JWT_TOKEN.JWT);

        // Formats the Private key to get away with the headers and whitespace characters
        privateKey = privateKey.replace("-----BEGIN RSA PRIVATE KEY-----", "");
        privateKey = privateKey.replace("-----END RSA PRIVATE KEY-----", "");
        privateKey = privateKey.replace("\\n","");
        privateKey = privateKey.replaceAll("\\s+", "");

        byte[] encodedKey = Base64.getDecoder().decode(privateKey);
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(encodedKey);

        KeyFactory kf = KeyFactory.getInstance(UltraConstants.JWT_TOKEN.RSA);
        PrivateKey privKey = kf.generatePrivate(keySpec);

        JwtBuilder jwtBuilder = Jwts.builder()
                .setIssuer(issuer)
                .setIssuedAt(now)
                .setHeader(headers)
                .signWith(signatureAlgorithm, privKey);

        // Assigns the expiration time of the JWT token
        Long expMillis = nowMillis + UltraConstants.JWT_TOKEN.EXPIRY_TIME;
        Date exp = new Date(expMillis);
        jwtBuilder.setExpiration(exp);

        retStr = jwtBuilder.compact();
        return retStr;
    }
}
```


###Generate Access Token
This class is used to generate the access token, which is used to fetch user resources.

```
import com.fasterxml.jackson.core.type.TypeReference;       
import com.fasterxml.jackson.databind.ObjectMapper;     
import com.sun.jersey.api.client.Client;        
import com.sun.jersey.api.client.WebResource;       
import <UltraConstants.java helper class>   
import <AccessTokenResponse.java helper class>   
import <Response.java helper class>

import java.io.IOException;

public class GenerateAccessToken {

    private String clientId;
    private String clientSecret;
    private static ObjectMapper objectMapper = new ObjectMapper();


    public GenerateAccessToken(String clientId, String clientSecret) {
        this.clientId = clientId;
        this.clientSecret = clientSecret;
    }
    /**
     * @param grantToken fetched from the UI
     * @param secureToken JWT token for server to server communication, generated by using GenerateJWTToken class
     * @return AccessTokenResponse containing accessToken and identityToken
     *
     * The Access Token contains information about which all scopes have been
     * approved as well as the merchant for which the token was generated.
     * For each partner, the identity token will remain constant per user.
     * Partner can use Identity Token as an index if they want.
     */

    public Response<AccessTokenResponse> generateAccessToken(String grantToken, String secureToken) throws IOException {

        //Creating client
        Client client = Client.create();

        //Specifying the resource url
        WebResource resource = client.resource(UltraConstants.ULTRA_SERVICE_PATHS.PLATFORM_PATH + UltraConstants.ULTRA_SERVICE_PATHS.ACCESS_TOKEN_PATH);

        //Building the resource and fetching response
        String response = resource.queryParam(UltraConstants.QUERY_PARAMETERS.GRANT_TOKEN, grantToken)
                .queryParam(UltraConstants.QUERY_PARAMETERS.CLIENT_ID, clientId)
                .queryParam(UltraConstants.QUERY_PARAMETERS.CLIENT_SECRET, clientSecret)
                .header(UltraConstants.HEADERS.CACHE_CONTROL, UltraConstants.HEADERS.NO_CACHE)
                .header(UltraConstants.HEADERS.CONTENT_TYPE, UltraConstants.HEADERS.JSON)
                .header(UltraConstants.HEADERS.SECURE_TOKEN, secureToken)
                .get(String.class);

        //Mapping the response
        Response<AccessTokenResponse> accessTokenResponse = objectMapper.readValue(response, new TypeReference<Response<AccessTokenResponse>>() {});
        return accessTokenResponse;

    }
}
```

###Get Bulk Resource
This class is used to fetch user resources for a particular user for a given partner.

```

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonArray;
import com.sun.jersey.api.client.Client;
import com.sun.jersey.api.client.WebResource;
import lombok.NoArgsConstructor;
import <UltraConstants.java helper class>   
import <Response.java helper class>

import java.io.IOException;
import java.util.List;
import java.util.Map;


@NoArgsConstructor  
public class GetBulkResource {

    private static ObjectMapper objectMapper = new ObjectMapper();

    /**
     * @param accessToken generated using GenerateAccessToken Class
     * @param secureToken JWT token for server to server communication, generated by using GenerateJWTToken class
     * @param resources list of scopes for a particular user for a given partner
     * Every granular resource which will be exposed to user as a permission
     * is represented as scope. Scopes have to be approved by user before our
     * merchant can hit it’s corresponding API using Access Token.
     * Scopes can be modified on the fly and every edit will lead to a new access token.
     * @return User resources for a given partner
     */

    public Map<String, Object> fetchBulkResource(String accessToken, String secureToken, List<String> resources) throws IOException {

        //Creating a client
        Client client = Client.create();

        //Specifying the resource url
        WebResource resource = client.resource(UltraConstants.ULTRA_SERVICE_PATHS.PLATFORM_PATH + UltraConstants.ULTRA_SERVICE_PATHS.BULK_RESOURCE_PATH);

        //Converting list<String> into JsonArray
        Gson gson = new GsonBuilder().create();
        JsonArray resourceList = gson.toJsonTree(resources).getAsJsonArray();

        //Building the resource and fetching response
        String response = resource.queryParam(UltraConstants.QUERY_PARAMETERS.ACCESS_TOKEN, accessToken)
                .header(UltraConstants.HEADERS.CONTENT_TYPE, UltraConstants.HEADERS.JSON)
                .header(UltraConstants.HEADERS.SECURE_TOKEN, secureToken)
                .post(String.class, resourceList.toString());

        //Mapping the response
        Response<Map<String, Object>> bulkResourceResponse = objectMapper.readValue(response, new TypeReference<Response<Map<String, Object>>>() {});
        return bulkResourceResponse.getResponse();
    }


}
```