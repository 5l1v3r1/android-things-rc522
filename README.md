# Android Things RC522 [![Bintray](https://img.shields.io/bintray/v/galarzaa90/maven/android-things-rc522.svg)](https://bintray.com/galarzaa90/maven/android-things-rc522) [![license](https://img.shields.io/github/license/Galarzaa90/android-things-rc522.svg)]() [![Android Things](https://img.shields.io/badge/android--things-0.2--devpreview-red.svg)](https://developer.android.com/things/preview/releases.html#developer_preview_2)

An Android Things libray to control RFID readers based on the RC522 reader.

Based on [pi-rc522](https://github.com/ondryaso/pi-rc522) by user **ondryaso**

### Features
* Detect MIFARE 1k tags (not tested in other tags)
* Authenticate, read and write to tags
* Change authentication keys and access bits (must be done manually)

### Planned features
* Increment, decrement, transfer and restore for value blocks
* Easier way of changing keys and access bits
* Helper functions

## Connections
The connections vary based on the [board](https://developer.android.com/things/hardware/developer-kits.html) used.

**RST** pin is configured programatically.

## Installing
This library is available at jCenter. To install add this to your module's build.gradle
```groovy


dependencies {
    compile 'com.galarzaa.android-things:rc522:0.2.1'
}
```

## Usage
_The use of interruptions is not supported yet._

The RC522 must be polled until a card is found, and then 
perform any operations you want.

Unfortunately, in Android, the UI thread shouldn't be blocked, so the polling has to be done on a 
separate thread e.g. AsyncTask, Runnable, etc.

To use the libary, a `SpiDevice` object must be passed in the constructor, along with a `Gpio` object for
the RST pin.

### Polling state
```java
import com.galarzaa.androidthings.Rc522;
public class MainActivty extends AppCompatActivity{
    private Rc522 mRc522;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        PeripheralManagerService pioService = new PeripheralManagerService();
        try {
            /* Names based on Raspberry Pi 3 */
            SpiDevice spiDevice = pioService.openSpiDevice("SPI0.0");
            Gpio resetPin = pioService.openGpio("BCM25");
            mRc522 = new Rc522(this, spiDevice, resetPin);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private void readRFid(){
        while(true){
            boolean success = mRc522.request();
            if(!success){
                continue;
            }
            success = mRc522.antiCollisionDetect();
            if(!success){
                continue;
            }
            byte[] uid = mRc522.getUid();
            mRc522.selectTag(uid);
            break;
        }
        // Factory Key A:
        byte[] key = {(byte)0xFF, (byte)0xFF, (byte)0xFF, (byte)0xFF, (byte)0xFF, (byte)0xFF};
        // Data that will be written
        byte[] newData = {0x0F,0x0E,0x0D,0x0C,0x0B,0x0A,0x09,0x08,0x07,0x06,0x05,0x04,0x03,0x02,0x01,0x00};
        // Get the address of the desired block
        byte block = Rc522.getBlockAddress(3, 2);
        //We need to authenticate the card, each sector can have a different key
        boolean result = rc522.authenticateCard(Rc522.AUTH_A, block, key);
        if (!result) {
            //Authentication failed
            return;
        }
        result = rc522.writeBlock(block, newData);
        if(!result){
            //Could not write, key might have permission to read but not write
            return;
        }
        //Buffer to hold read data
        byte[] buffer = new byte[16];
        //Since we're still using the same block, we don't need to authenticate again
        result = rc522.readBlock(block, buffer);
        if(!result){
            //Could not read card
            return;
        }
        //Stop crypto to allow subsequent readings
        rc522.stopCrypto();
            
        
    }
}
```

## Contributing
This library is still in development, suggestions, improvements and fixes are welcome. Please 
submit a **pull request**

## Resources
* [This library's javadoc](https://galarzaa90.github.io/android-things-rc522/com/galarzaa/androidthings/Rc522.html#constructor.detail)
* [MFRC522 product data sheet, pdf, 95 pages](http://www.nxp.com/docs/en/data-sheet/MFRC522.pdf)
* [MIFARE Classic EV1 1K tags data product data sheet, pdf, 20 pages](http://www.nxp.com/docs/en/data-sheet/MFRC522.pdf)
