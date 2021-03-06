---
title: Encoder Code Examples
author: windows-driver-content
description: Encoder Code Examples
MS-HAID:
- 'encodeds\_75b74f82-f7a7-4ea3-9e6a-989affddc628.xml'
- 'stream.encoder\_code\_examples'
MSHAttr:
- 'PreferredSiteName:MSDN'
- 'PreferredLib:/library/windows/hardware'
ms.assetid: cbe773ad-2222-4d62-8e1e-6d47418a3e7c
keywords: ["variable bit rates WDK encoder", "encoder devices WDK AVStream", "AVStream WDK , encoder devices", "uncompressed data streams WDK AVStream", "encoded streams WDK AVStream", "audio encoder devices WDK AVStream", "video encoder devices WDK AVStream", "ENCAPIPARAM_BITRATE_MODE", "ENCAPIPARAM_BITRATE", "bit rates WDK encoder", "registry WDK encoder"]
---

# Encoder Code Examples


The following code examples are based on the [AVStream Simulated Hardware Sample Driver (AVSHwS)](http://go.microsoft.com/fwlink/p/?linkid=256083) in the MSDN Code Gallery. They demonstrate the following:

-   How to specify the encoder's supported bit rates

-   How to specify the bit rate encoding modes supported by an encoder

-   How to specify metadata values at run time under the encoder device's *Device Parameters\\Capabilities* registry key

### **Implementing Supported Bit Rates**

The following code snippets demonstrate how to implement support for the [ENCAPIPARAM\_BITRATE](https://msdn.microsoft.com/library/windows/hardware/ff559520) property. Use a [**KSPROPERTY\_STEPPING\_LONG**](https://msdn.microsoft.com/library/windows/hardware/ff565631) structure to specify a stepping granularity of 400 bits per second (bps) with a 400-bps lower-bound and 4,000,000-bps upper-bound.

```
const KSPROPERTY_STEPPING_LONG BitRateRanges [] = {
    {
        400,
        0,
        400,
        4000000
    }
};
```

If you access the encoder filter's property page by right-clicking on the filter in a tool such as GraphEdit, you will see the **Bit Rate** slider bar where these values are used.

Next, specify the default encoding bit rate of the encoder filter when an instance of it is created. Note that the data type used is a ULONG that corresponds to the property value type required by the ENCAPIPARAM\_BITRATE property. This value is the default encoding "Bit Rate" that is displayed in the encoder's property page:

```
const ULONG BitRateValues [] = {
    1000000
};
```

Specify the list of legal ranges and a default value for the ENCAPIPARAM\_BITRATE property:

```
 const KSPROPERTY_MEMBERSLIST BitRateMembersList [] = {
    {
        {
            KSPROPERTY_MEMBER_STEPPEDRANGES,
            sizeof (BitRateRanges),
            SIZEOF_ARRAY (BitRateRanges),
            0
        },
        BitRateRanges
    },
    {
        {
            KSPROPERTY_MEMBER_VALUES,
            sizeof (BitRateValues),
            SIZEOF_ARRAY (BitRateValues),
            KSPROPERTY_MEMBER_FLAG_DEFAULT
        },
        BitRateValues
    }
};
```

```
 const KSPROPERTY_VALUES BitRateValuesSet = {
    {
        STATICGUIDOF (KSPROPTYPESETID_General),
        VT_UI4,
        0
    },
    SIZEOF_ARRAY (BitRateMembersList),
    BitRateMembersList
};
```

Specify the single property defined for the ENCAPIPARAM\_BITRATE property set:

```
DEFINE_KSPROPERTY_TABLE(ENCAPI_BitRate) {
    DEFINE_KSPROPERTY_ITEM (
        0,
        GetBitRateHandler, //Get-property handler supported
        sizeof (KSPROPERTY),
        sizeof (ULONG),
        SetBitRateHandler, //Set-property handler supported
        &amp;BitRateValuesSet,
        0,
        NULL,
        NULL,
        sizeof (ULONG)
        )
};
```

**Note**   The *get*-property handler returns the encoding bit rate, and the *Set*-property handler must test that the incoming passed-in value is valid before using it.

 

### **Implementing Supported Encoding Bit Rate Modes**

The following code snippets demonstrate how to implement support for the [ENCAPIPARAM\_BITRATE\_MODE](https://msdn.microsoft.com/library/windows/hardware/ff559524) property.

Define the encoding modes supported by the encoder:

```
 const VIDEOENCODER_BITRATE_MODE BitRateModeValues [] = {
    ConstantBitRate,
    VariableBitRateAverage
};
```

Specify the default encoding bit rate mode as average variable bit rate:

```
const VIDEOENCODER_BITRATE_MODE BitRateModeDefaultValues [] = {
    VariableBitRateAverage
};
```

Specify the list of legal ranges and default value for the ENCAPIPARAM\_BITRATE\_MODE property:

```
const KSPROPERTY_MEMBERSLIST BitRateModeMembersList [] = {
    {
        {
            KSPROPERTY_MEMBER_VALUES,
            sizeof (BitRateModeValues),
            SIZEOF_ARRAY (BitRateModeValues),
            0
        },
        BitRateModeValues
    },
    {
        {
            KSPROPERTY_MEMBER_VALUES,
            sizeof (BitRateModeDefaultValues),
            SIZEOF_ARRAY (BitRateModeDefaultValues),
            KSPROPERTY_MEMBER_FLAG_DEFAULT
        },
        BitRateModeDefaultValues
    }
};

const KSPROPERTY_VALUES BitRateModeValuesSet = {
    {
        STATICGUIDOF (KSPROPTYPESETID_General),
        VT_I4,
        0
    },
    SIZEOF_ARRAY (BitRateModeMembersList),
    BitRateModeMembersList
};
```

Specify the single property defined for the ENCAPIPARAM\_BITRATE\_MODE property set:

```
DEFINE_KSPROPERTY_TABLE(ENCAPI_BitRateMode) {
    DEFINE_KSPROPERTY_ITEM (
        0,
        GetBitRateModeHandler, //Get-property handler supported
        sizeof (KSPROPERTY),
        sizeof (VIDEOENCODER_BITRATE_MODE),
        SetBitRateModeHandler, //Set-property handler supported
        &amp;BitRateModeValuesSet,
        0,
        NULL,
        NULL,
        sizeof (VIDEOENCODER_BITRATE_MODE)
        )
};
```

**Note**   The *get*-property handler should return the encoding bit rate mode, and the *Set*-property handler must test that the incoming passed-in value is valid before using it.

 

The property sets are then specified as the [**KSFILTER\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff562553) structure's automation table.

```
DEFINE_KSPROPERTY_SET_TABLE(PropertyTable) {
    DEFINE_KSPROPERTY_SET(
        &amp;ENCAPIPARAM_BITRATE_MODE,
        SIZEOF_ARRAY (ENCAPI_BitRateMode),
        ENCAPI_BitRateMode,
        0,
        NULL
        ),
    DEFINE_KSPROPERTY_SET(
        &amp;ENCAPIPARAM_BITRATE,
        SIZEOF_ARRAY (ENCAPI_BitRate),
        ENCAPI_BitRate,
        0,
        NULL
        )
};

DEFINE_KSAUTOMATION_TABLE(FilterTestTable) {
    DEFINE_KSAUTOMATION_PROPERTIES(PropertyTable),
    DEFINE_KSAUTOMATION_METHODS_NULL,
    DEFINE_KSAUTOMATION_EVENTS_NULL
};

const 
KSFILTER_DESCRIPTOR 
FilterDescriptor = {
    ...,
    &amp;FilterTestTable, // Automation Table
    ...,
    ...
};
```

### <a href="" id="specifying-the-encoder-s-capabilities-in-the-registry"></a>**Specifying the Encoder's Capabilities in the Registry**

The following code sample demonstrates how to create a *Capabilities* registry key under the *Device Parameters* registry key, and how to create and specify sub keys and values under the *Capabilities* key. Execute this code when the driver initializes.

**Note:** The following code assumes the presence of a single hardware encoder per physical device. If your hardware contains more than one encoder then you must iterate through the list returned in the call to the **IoGetDeviceInterfaces** function and register the capabilities for each encoder.

```
/**************************************************************************
CreateDwordValueInCapabilityRegistry()

IN Pdo: PhysicalDeviceObject
IN categoryGUID: Category GUID eg KSCATEGORY_CAPTURE

1. Get Symbolic name for interface
2. Open registry key for storing information about a 
   particular device interface instance
3. Create Capabilities key under "Device Parameters" key
4. Create a DWORD value "TestCapValueDWORD" under Capabilities

Must be running at IRQL = PASSIVE_LEVEL in the context of a system thread
**************************************************************************/
NTSTATUS CreateDwordValueInCapabilityRegistry(IN PDEVICE_OBJECT pdo, IN GUID categoryGUID)

{

    // 1. Get Symbolic name for interface
    // pSymbolicNameList can contain multiple strings if pdo is NULL. 
    // Driver should parse this list of string to get 
    // the one corresponding to current device interface instance. 
    PWSTR  pSymbolicNameList = NULL;

    NTSTATUS ntStatus = IoGetDeviceInterfaces(
        &amp;categoryGUID,
        pdo,
        DEVICE_INTERFACE_INCLUDE_NONACTIVE,
        &amp;pSymbolicNameList);
    if (NT_SUCCESS(ntStatus) &amp;&amp; (NULL != pSymbolicNameList))
    {
        HANDLE hDeviceParametersKey = NULL;
        UNICODE_STRING symbolicName;

        // 2. Open registry key for storing information about a 
        // particular device interface instance
        RtlInitUnicodeString(&amp;symbolicName, pSymbolicNameList);
        ntStatus = IoOpenDeviceInterfaceRegistryKey(
            &amp;symbolicName,
            KEY_READ|KEY_WRITE,
            &amp;hDeviceParametersKey);
        if (NT_SUCCESS(ntStatus))
        {
            OBJECT_ATTRIBUTES objAttribSubKey;
            UNICODE_STRING subKey;
 
            // 3. Create Capabilities key under "Device Parameters" key
            RtlInitUnicodeString(&amp;subKey,L"Capabilities");
            InitializeObjectAttributes(&amp;objAttribSubKey,
                &amp;subKey,
                OBJ_KERNEL_HANDLE,
                hDeviceParametersKey,
                NULL);
 
            HANDLE hCapabilityKeyHandle = NULL;
 
            ntStatus = ZwCreateKey(&amp;hCapabilityKeyHandle,
                    KEY_READ|KEY_WRITE|KEY_SET_VALUE,
                    &amp;objAttribSubKey,
                    0,
                    NULL,
                    REG_OPTION_NON_VOLATILE,
                    NULL);
            if (NT_SUCCESS(ntStatus))
            {
                OBJECT_ATTRIBUTES objAttribDwordKeyVal;
                UNICODE_STRING subValDword;
 
                // 4. Create a DWORD value "TestCapValueDWORD" under Capabilities 
                RtlInitUnicodeString(&amp;subValDword,L"TestCapValueDWORD");
 
                ULONG data = 0xaaaaaaaa;
 
                ntStatus = ZwSetValueKey(hCapabilityKeyHandle,&amp;subValDword,0,REG_DWORD,&amp;data,sizeof(ULONG));
                ZwClose(hCapabilityKeyHandle);
            }
        }
        ZwClose(hDeviceParametersKey);
        ExFreePool(pSymbolicNameList);
    }
 
    return ntStatus;
}
```

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Bstream\stream%5D:%20Encoder%20Code%20Examples%20%20RELEASE:%20%288/23/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")


