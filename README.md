Code that can help to debug metadata_data_t struct used by qualcomm camera code.

It is located in QCamera2/stack/common/cam_intf.h

Main idea is to compare output from functions get_pointer_of() and get_size_of() from binary libmmcamera2_mct.so, (liboemcamera.so for older devices) with values defined in oss header.

To get started, you have to make a list of fields of metadata_data_t using special macro PRINT(), like 

```
    PRINT(CAM_INTF_META_FACE_DETECTION,         pMetadata);
    PRINT(CAM_INTF_META_FACE_RECOG,             pMetadata);
```
The Python script helps you generating this list, just make sure to make it find cam_intf.h header to generate a text file called list.txt

Next build it and copy get_offsets binary to /system/vendor/lib on your device where libmmcamera2_mct.so located.

Run it and it will produce output like this 

```
BLOB: CAM_INTF_META_AUTOFOCUS_DATA index=19 pointer=8784 size=56
OSS : CAM_INTF_META_AUTOFOCUS_DATA index=19 pointer=8784 size=56
FIXME!!!
BLOB: CAM_INTF_META_CDS_DATA index=203 pointer=310378496 size=0
OSS : CAM_INTF_META_CDS_DATA index=203 pointer=8840 size=68
BLOB: CAM_INTF_PARM_UPDATE_DEBUG_LEVEL index=175 pointer=8908 size=4
OSS : CAM_INTF_PARM_UPDATE_DEBUG_LEVEL index=175 pointer=8908 size=4
```

Explanations:
* BLOB means values found in libmmcamera2_mct.so
* OSS means values from your current cam_intf.h
* FIXME means fields that different by offset or by size. We differ by three cases:
	1.Pointer in BLOB is higher: either shift the INCLUDE instructions with a volatile char array by the amount of bytes which is differing
	2.Size is bigger in BLOB: if the pointer is a struct  make sure to add padding bytes to make it bigger to match size
	3.Some BLOB values will be size=0 and big pointer, it means this field not used in blob (but thats *not* mean you have to remove it from oss HAL). Still work in progress to find out what's best to be done
