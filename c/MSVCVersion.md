 ΢���1998�귢��Visual C++ 6.0��������ÿ2��һ��IDE���ٶȷ����µĿ���ƽ̨�����������ʷ�ϸ���VS�İ汾��


|IDE|����ʱ��|���߼��汾|MSC_VER|MSVC++|ϵͳ֧��|ʹ��Ƶ��|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|Visual C++6.0|1998|V60|1200|MSVC++ 6.0|XP��Win7|��Ƶ|
|Visual Studio 2002|2002|V70|1300|MSVC++ 7.0|XP��Win7|��Ƶ|
|Visual Studio 2003|2003|V71|1310|MSVC++ 7.1|XP��Win7|��Ƶ|
|Visual Studio 2005|2005|V80|1400|MSVC++ 8.0|XP��Win7|��Ƶ|
|Visual Studio 2008|2008|V90|1500|MSVC++ 9.0|XP��Win7|��Ƶ|
|Visual Studio 2010|2010|V100|1600|MSVC++ 10.0|XP��Win7|��Ƶ|
|Visual Studio 2012|2012|V110|1700|MSVC++ 11.0|XP��Win7|��Ƶ|
|Visual Studio 2013|2013|V120|1800|MSVC++ 12.0|Win10��Win7|��Ƶ|
|Visual Studio 2015|2015|V140|1900|MSVC++ 14.0|Win10��Win7|��Ƶ|
|Visual Studio 2017|2017|V141|1910|MSVC++ 14.1|Win10��Win7|��Ƶ|
|Visual Studio 2019|2019|V142|1920|MSVC++ 14.2|Win10��Win7|��Ƶ|
|Visual Studio 2022|2022|V143|1930|MSVC++ 14.3|Win10��Win7|��Ƶ|

������ı���У����Կ�����XPϵͳ���֧�ֵ�VS2012��Win7֧��֧��VC++6 ~ VS2019��Win10֧��VS2013~VS2022��

��õ��ǣ�VC++6.0 ��VS2008��VS2010��VS2013��
������ͨ��ʹ��VS�Ĺ��߼��汾��V60~V142����ʾ����VS��������IDE�ķ���ʱ�䡣




    MSVC++  14 . 0  _ MSC _ VER  == 1900 (Visual Studio 2015)
    MSVC++  12. 0 _ MSC_ VER == 1800 (Visual Studio 2013)
    MSVC++ 11. 0 _ MSC_ VER == 1700 (Visual Studio 2012)
    MSVC++ 10. 0 _ MSC_ VER == 1600 (Visual Studio 2010)
    MSVC++ 9. 0  _ MSC_ VER == 1500 (Visual Studio 2008)
    MSVC++ 8. 0  _ MSC_ VER == 1400 (Visual Studio 2005)
    MSVC++ 7.1  _ MSC_ VER == 1310 (Visual Studio 2003)
    MSVC++ 7. 0  _ MSC_ VER == 1300
    MSVC++ 6. 0  _ MSC_ VER == 1200
    MSVC++ 5. 0  _ MSC_ VER == 1100
           
    c:program files (x86) microsoft visual studio 11.0vcbin>cl /?
    Microsoft (R) C/C++ Optimizing Compiler Version 17.00.50727.1 for x86
    .....
           
    #if (_ MSC_ VER == 1500)
       // ... Do VC9/Visual Studio 2008 specific stuff
    #elif (_ MSC_ VER == 1600)
       // ... Do VC10/Visual Studio 2010 specific stuff
    #elif (_ MSC_ VER == 1700)
       // ... Do VC11/Visual Studio 2012 specific stuff
    #endif
           
    #if (_ MSC_ VER >= 1500 && _ MSC_ VER <= 1600)
       // ... Do VC9/Visual Studio 2008 specific stuff
    #endif
           
    //******************************************************************************
    // Automated platform detection
    //******************************************************************************
     
    // _WIN32 is used by
    // Visual C++
    #ifdef _WIN32
    #define __NT__
    #endif
     
    // Define __MAC__ platform indicator
    #ifdef macintosh
    #define __MAC__
    #endif
     
    // Define __OSX__ platform indicator
    #ifdef __APPLE__
    #define __OSX__
    #endif
     
    // Define __WIN16__ platform indicator
    #ifdef _Windows_
    #ifndef __NT__
    #define __WIN16__
    #endif
    #endif
     
    // Define Windows CE platform indicator
    #ifdef WIN32_PLATFORM_HPCPRO
    #define __WINCE__
    #endif
     
    #if (_WIN32_WCE == 300) // for Pocket PC
    #define __POCKETPC__
    #define __WINCE__
    //#if (_WIN32_WCE == 211) // for Palm-size PC 2.11 (Wyvern)
    //#if (_WIN32_WCE == 201) // for Palm-size PC 2.01 (Gryphon)  
    //#ifdef WIN32_PLATFORM_HPC2000 // for H/PC 2000 (Galileo)
    #endif

