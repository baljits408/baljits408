RxConnect
An Android Library to POST JSON and normal GET/POST using Rxjava.

Add this line to your dependencies in your apps build.gradle

compile 'mohitbadwal.rxconnect:rxconnect:1.0.42'

Usage
Make new RxConnect object

RxConnect rxConnect=new RxConnect(context);
Set Caching

//by default caching is enabled for better performance
rxConnect.setCachingEnabled(false);
//call this method before setting parameters if you don't want caching
Set Add To RequestQueue

//by default Requests are added to queue for better performance , helps when device goes offline
rxConnect.setAddToQueue(false);
//call this method before setting parameters if you don't want requests being added to queue
Set Parameter to send

rxConnect.setParam("phone","9999999999");
// setParam(keys,values)
rxConnect.setParam("password","enteredpassword");
Use execute method to perform operation

      //all override methods run on ui thread
      //use RxConnect.GET for GET
      //use RxConnect.POST for POST
      //use RxConnect.JSON_POST for POST JSON data
        rxConnect.execute(yoururl,RxConnect.GET, new RxConnect.RxResultHelper() {
            @Override
            public void onResult(String result) {
              //do something on result
            }

            @Override
            public void onNoResult() {
                //do something
            }

            @Override
            public void onError(Throwable throwable) {
               //do somenthing on error
            }

        });
About
An Android Library to POST JSON and normal GET/POST using Rxjava. It's a networking library.

Topics
post-json network rxjava android-library requests
Resources
 Readme
 Activity
Stars
 3 stars
Watchers
 1 watching
Forks
 0 forks
Report repository
Releases 1
Added Caching to RxConnect
Latest
on May 23, 2016
Packages
No packages published
Languages
Java
100.0%
Footer
© 2025 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
