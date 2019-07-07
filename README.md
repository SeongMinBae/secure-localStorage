# secure-localStorage
Secure localStorage data with AES encryption and data compression that allows one to encrypt/decrypt the data being stored🔒

What Secure-localStorage Looks Like
===================================
    <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.2/rollups/aes.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.2/rollups/pbkdf2.js"></script>

  
    <script>
    var keySize = 256;
    var ivSize = 128;
    var iterations = 100;

    var secret_key = "Secret Password(서버단에서 제공)";

    function encrypt (msg, pass) {
      var salt = CryptoJS.lib.WordArray.random(128/8);
      
      var key = CryptoJS.PBKDF2(pass, salt, {
          keySize: keySize/32,
          iterations: iterations
        });

      var iv = CryptoJS.lib.WordArray.random(128/8);
      
      var encrypted = CryptoJS.AES.encrypt(msg, key, { 
        iv: iv, 
        padding: CryptoJS.pad.Pkcs7,
        mode: CryptoJS.mode.CBC
        
      });
      
      var transitmessage = salt.toString()+ iv.toString() + encrypted.toString();
      return transitmessage;
    }

    function decrypt (transitmessage, pass) {
      var salt = CryptoJS.enc.Hex.parse(transitmessage.substr(0, 32));
      var iv = CryptoJS.enc.Hex.parse(transitmessage.substr(32, 32))
      var encrypted = transitmessage.substring(64);
      
      var key = CryptoJS.PBKDF2(pass, salt, {
          keySize: keySize/32,
          iterations: iterations
        });

      var decrypted = CryptoJS.AES.decrypt(encrypted, key, { 
        iv: iv, 
        padding: CryptoJS.pad.Pkcs7,
        mode: CryptoJS.mode.CBC
        
      })
      return decrypted;
    }

    secureStorage = {
        setItem : function(key, value){
            try{
              var encStr = encrypt(value, secret_key);
              localStorage.setItem(key,encStr);
              return encStr;
            }catch(e){
              return false;
            }
        },
        getItem : function(key){
            try{
              var decStr = decrypt(localStorage.getItem(key),secret_key);
              return decStr.toString(CryptoJS.enc.Utf8);
            }catch(e){
              return false;
            }
        },
        removeItem : function(key){
            try{
                localStorage.removeItem(key);
                return true;    
            }catch(e){
                return false;
            }
        },
        clear : function(){
            try{
                localStorage.clear();
                return true;
            }catch(e){
                return false;
            }
        }
    }
 
    </script>
  
  

Usage
==========
    secureStorage.setItem('mydog', 'tantan'); // setItem(key, value);
    
    secureStorage.getItem('mydog'); // getItem(key);
    
    secureStorage.removeItem('mydog'); // removeItem(key);
    
    secureStorage.clear(); // clear();


