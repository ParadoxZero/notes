## How to use

```bash
python sig_walker 
```

## Script

```python

import base64
import io
import pypdfium2.raw as pdfium
import asn1
import ctypes
import binascii

tag_name = {
    asn1.Numbers.Boolean: "BOOLEAN",
    asn1.Numbers.Integer: "INTEGER",
    asn1.Numbers.BitString: "BIT STRING",
    asn1.Numbers.OctetString: "OCTET STRING",
    asn1.Numbers.Null: "NULL",
    asn1.Numbers.ObjectIdentifier: "OBJECT",
    asn1.Numbers.PrintableString: "PRINTABLESTRING",
    asn1.Numbers.IA5String: "IA5STRING",
    asn1.Numbers.UTCTime: "UTCTIME",
    asn1.Numbers.GeneralizedTime: "GENERALIZED TIME",
    asn1.Numbers.Enumerated: "ENUMERATED",
    asn1.Numbers.Sequence: "SEQUENCE",
    asn1.Numbers.Set: "SET"
}
class_name = {
    asn1.Classes.Universal: "U",
    asn1.Classes.Application: "A",
    asn1.Classes.Context: "C",
    asn1.Classes.Private: "P"
}
object_name = {
    "1.2.840.113549.1.1.1": "rsaEncryption",
    "1.2.840.113549.1.1.5": "sha1WithRSAEncryption",
    "1.3.6.1.5.5.7.1.1": "authorityInfoAccess",
    "2.5.4.3": "commonName",
    "2.5.4.4": "surname",
    "2.5.4.5": "serialNumber",
    "2.5.4.6": "countryName",
    "2.5.4.7": "localityName",
    "2.5.4.8": "stateOrProvinceName",
    "2.5.4.9": "streetAddress",
    "2.5.4.10": "organizationName",
    "2.5.4.11": "organizationalUnitName",
    "2.5.4.12": "title",
    "2.5.4.13": "description",
    "2.5.4.42": "givenName",
    "2.5.29.21": "reasonCode",
    "2.5.29.20": "cRLNumber",
    "1.2.840.113549.1.9.1": "emailAddress",
    "1.2.840.113549.1.7.2": "signedData",
    "1.2.840.113549.1.7.1": "data",
    "1.2.840.113549.1.9.1": "emailAddress",
    "1.2.840.113549.1.9.3": "contentType",
    "1.2.840.113549.1.9.4": "messageDigest",
    "1.2.840.113549.1.1.11": "sha256WithRSAEncryption",
    "1.2.840.113583.1.1.10": "PPLKLiteCredential", 
    "2.5.29.14": "X509v3 Subject Key Identifier",
    "2.5.29.15": "X509v3 Key Usage",
    "2.5.29.16": "X509v3 Private Key Usage Period",
    "2.5.29.17": "X509v3 Subject Alternative Name",
    "2.5.29.18": "X509v3 Issuer Alternative Name",
    "2.5.29.19": "X509v3 Basic Constraints",
    "2.5.29.30": "X509v3 Name Constraints",
    "2.5.29.31": "X509v3 CRL Distribution Points",
    "2.5.29.32": "X509v3 Certificate Policies Extension",
    "2.5.29.33": "X509v3 Policy Mappings",
    "2.5.29.35": "X509v3 Authority Key Identifier",
    "2.5.29.36": "X509v3 Policy Constraints",
    "2.5.29.37": "X509v3 Extended Key Usage",
    "2.16.840.1.101.3.4.2.1": "SHA-256",
    "1.2.840.113549.1.1.12": "sha384WithRSAEncryption",
    "1.2.840.113549.1.9.16.2.47": "id-aa-signingCertificateV2",
    "1.2.840.113583.1.1.8": "revocationInfoArchival",
    "1.2.840.113549.1.9.5": "signingTime",
}


def get_tag_name(identifier):
    if identifier in tag_name:
        return tag_name[identifier]
    return '{:#02x}'.format(identifier)


def get_class_name(identifier):
    if identifier in class_name:
        return class_name[identifier]
    raise ValueError('Illegal class: {:#02x}'.format(identifier))


def get_object_name(identifier):
    if identifier in object_name:
        return object_name[identifier]
    return identifier


def value_to_string(tag_number, value):
    if tag_number == asn1.Numbers.ObjectIdentifier:
        return get_object_name(value)
    elif isinstance(value, bytes):
        return str(binascii.hexlify(value, ",").upper())
    elif isinstance(value, str):
        return value
    else:
        return repr(value)


def get_signature_content(pdf_path, index):
    doc = pdfium.FPDF_LoadDocument(pdf_path.encode("utf-8"), None)
    if not doc:
        raise ValueError("Failed to load document")
    try:
        count = pdfium.FPDF_GetSignatureCount(doc)
        print("Number of sig found: {}".format(count))
        signature_dict = pdfium.FPDF_GetSignatureObject(doc, index)
        if signature_dict:
            subfilter_buffer = ctypes.create_string_buffer(256)
            pdfium.FPDFSignatureObj_GetSubFilter(
                signature_dict, subfilter_buffer, 256)
            print("Subfilter: {}".format(subfilter_buffer.value.decode("utf-8")))
            array_length = pdfium.FPDFSignatureObj_GetContents(
                signature_dict, None, 0)
            buffer = ctypes.create_string_buffer(array_length)
            contents = pdfium.FPDFSignatureObj_GetContents(
                signature_dict, buffer, array_length)
            return buffer.raw
        else:
            raise ValueError("Failed to get signature object")
    finally:
        pdfium.FPDF_CloseDocument(doc)


def get_byte_range(pdf_path, index):
    doc = pdfium.FPDF_LoadDocument(pdf_path.encode("utf-8"), None)
    if not doc:
        raise ValueError("Failed to load document")
    try:
        count = pdfium.FPDF_GetSignatureCount(doc)
        print("Number of sig found: {}".format(count))
        signature_dict = pdfium.FPDF_GetSignatureObject(doc, index)
        if signature_dict:

            array_length = pdfium.FPDFSignatureObj_GetByteRange(
                signature_dict, None, 0)
            buffer = (ctypes.c_long * 4)()
            buffer_length = ctypes.c_ulong(16)
            contents = pdfium.FPDFSignatureObj_GetByteRange(
                signature_dict, buffer, buffer_length)

            byte_range = list(buffer)[:buffer_length.value]
            return byte_range
        else:
            raise ValueError("Failed to get signature object")
    finally:
        pdfium.FPDF_CloseDocument(doc)


def empty_signature_content(pdf_path, byte_range):
    if not pdf_path.endswith(".pdf"):
        raise ValueError("Not a pdf file")
    new_path = pdf_path[:-4] + "_empty.pdf"
    print("New path: {}".format(new_path))
    import shutil
    shutil.copy(pdf_path, new_path)
    with open(new_path, 'rb+') as file:
        start = byte_range[1] + 1
        end = byte_range[2]
        file.seek(start)      # Seek to the starting offset
        content_length = end - start -1   # Calculate the length of the content to replace
        file.write(b'\x00'*content_length)      # Write the new content


def dump_asn1(content, output_stream, indent=0):
    while not content.eof():
        tag = content.peek()
        if tag.typ == asn1.Types.Primitive:
            output_stream.write(' ' * indent)
            tag, value = content.read()
            if tag.nr == 0x0 and tag.cls == asn1.Classes.Universal and indent == 0:
                output_stream.write('[{}] {}: {}\n'.format(tag.cls, tag.nr, value))
                break
            output_stream.write('[{}] {}: {}\n'.format(get_class_name(tag.cls),
                                                       get_tag_name(tag.nr), value_to_string(tag.nr, value)))
        elif tag.typ == asn1.Types.Constructed:
            output_stream.write(' ' * indent)
            output_stream.write('[{}] {}\n'.format(get_class_name(tag.cls),
                                                   get_tag_name(tag.nr)))
            output_stream.write(' ' * indent)
            output_stream.write('{\n')
            content.enter()
            dump_asn1(content, output_stream, indent + 2)
            content.leave()
            output_stream.write(' ' * indent)
            output_stream.write('}\n')
        else:
            raise ValueError('Invalid tag')
    output_stream.write(' ' * indent)
    output_stream.write('\n')

def print_object_ids(content, output_stream):
    while not content.eof():
        tag = content.peek()
        if tag.typ == asn1.Types.Primitive:
            if tag.nr == 0x0 and tag.cls == asn1.Classes.Universal:
                break
            tag, value = content.read()
            if tag.nr == asn1.Numbers.ObjectIdentifier:
                if value not in object_name:
                    output_stream.write('{}\n'.format(value))
                else:
                    output_stream.write('[{}] {}\n'.format(
                        object_name[value], value))
        if tag.typ == asn1.Types.Constructed:
            content.enter()
            print_object_ids(content, output_stream)
            content.leave()


def find_oid(content, oid):
    while not content.eof():
        tag = content.peek()
        if tag.typ == asn1.Types.Primitive:
            if tag.nr == 0x0 and tag.cls == asn1.Classes.Universal:
                break
            tag, value = content.read()
            if tag.nr == asn1.Numbers.ObjectIdentifier:
                if value == oid:
                    return value
        if tag.typ == asn1.Types.Constructed:
            content.enter()
            oid = find_oid(content, oid)
            if oid:
                return oid
            content.leave()
    return None

# convert hexadecimel string to byte array


def to_byte_array(hex_string):
    return bytearray.fromhex(hex_string)

def OID_encoder(oid_string):
    encoder = asn1.Encoder()
    encoder.start()
    encoder.write(oid_string, asn1.Numbers.ObjectIdentifier)
    encoded_bytes = encoder.output()
    hex_string = encoded_bytes.hex()
    for i in range(4, len(hex_string), 2):
        print("0x{}".format(hex_string[i:i+2]), end=",")

def OID_decoder(hex_string):
    decoder = asn1.Decoder()
    print(binascii.a2b_hex(hex_string))
    decoder.start(binascii.a2b_hex(hex_string))
    print(decoder.read( asn1.Numbers.ObjectIdentifier))

def sign_content():
    pfx_file = sys.argv[2]
    pfx_password = sys.argv[3]
    content = sys.argv[4]

    from binascii import unhexlify
    # Convert hex string to bytes
    data_to_sign = unhexlify(content)

    # Load private key and certificate from PFX file
    from cryptography.hazmat.primitives.serialization import pkcs12

    with open(pfx_file, "rb") as f:
        private_key, _, _ = pkcs12.load_key_and_certificates(f.read(), pfx_password.encode())
        # Sign the content

        from cryptography.hazmat.primitives import hashes
        from cryptography.hazmat.primitives.asymmetric import padding
        signature = private_key.sign(
            data=data_to_sign,
            algorithm=hashes.SHA512_256(),
            padding=padding.PKCS1v15(), #padding.PSS(mgf=padding.MGF1(hashes.SHA512_256()), salt_length=0)
        )
        print (signature.hex())

def verify_signature():
    from cryptography.hazmat.primitives.serialization import pkcs12
    from cryptography.hazmat.primitives import hashes
    from cryptography.hazmat.primitives.asymmetric import padding
    from binascii import unhexlify

    pfx_file = sys.argv[2]
    pfx_password = sys.argv[3]
    content = sys.argv[4]
    signature = sys.argv[5]

    # Convert hex string to bytes
    data_to_verify = unhexlify(content)
    signature = unhexlify(signature)

    # Load private key and certificate from PFX file
    with open(pfx_file, "rb") as f:
        private_key, certificate, _ = pkcs12.load_key_and_certificates(f.read(), pfx_password.encode())

    # Verify the signature
    public_key = certificate.public_key()
    public_key.verify(
        signature=signature,
        data=data_to_verify,
        padding=padding.PKCS1v15(),
        algorithm=hashes.SHA512_256(),
    )

def process_pdf_commands(command):
    static_content = "<>"

    file = sys.argv[2]
    index = int(sys.argv[3])
    output = None
    if file == "static":
        file = static_content
    if file.endswith(".pdf"):
        content = get_signature_content(file, index)
    elif file.endswith(".bin"):
        with open(file, "rb") as f:
            content = f.read()
    elif file.find(",") != -1:
        bin_array = []
        for i in file.split(","):
            bin_array.append(int(i, 16))
        content = bytes(bin_array)
    else:
        content = binascii.a2b_hex(file)
    print("Content size: {}".format(len(content)))

    try:
        if command == "id":
            decoder = asn1.Decoder()
            decoder.start(content)
            stream = io.StringIO()
            print_object_ids(decoder, stream)
            print(stream.getvalue())

        elif command == "print":
            for i in to_byte_array(content.hex()):
                print("{}".format(i), end=",")
        elif command == "hex":
            print(content.hex())
        
        elif command == "base64":
            print(binascii.b2a_base64(content))
        
        elif command == "bin":
            # Print Binary string from bytes
            print("".join(format(x, "b") for x in content))

        elif command == "dump":
            decoder = asn1.Decoder()
            decoder.start(content)
            stream = io.StringIO()
            dump_asn1(decoder, stream)
            try:
                output = sys.argv[4]
                with open(output, "w") as f:
                    f.write(stream.getvalue())
            except IndexError:
                print(stream.getvalue())

        elif command == "decode":
            from asn1crypto import cms
            info = cms.SignedData.load(content)
            import json
            from collections import OrderedDict

            def serialize_list(data, dict_serializer):
                _data = []
                for v in data:
                    if isinstance(v, OrderedDict):
                        _data.append(dict_serializer(v))
                    elif isinstance(v, list):
                        _data.append(serialize_list(v, dict_serializer))
                    elif isinstance(v, bytes):
                        _data.append(v.hex())
                    else:
                        _data.append(str(v))
                return _data
            
            def serialize(data):
                _data = {}
                for k, v in data.items():
                    if isinstance(v, OrderedDict):
                        _data[k] = serialize(v)
                    elif isinstance(v, list):
                        _data[k] = serialize_list(v, serialize)
                    elif isinstance(v, bytes):
                        _data[k] = v.hex()
                    else:
                        _data[k] = str(v)
                return _data
            print(json.dumps(serialize(info.native)))

        elif command == "oid":
            oid = find_oid(decoder, sys.argv[4])
            if oid:
                print(oid)
            else:
                print("Not found")

        elif command == "byte_range":
            byte_range = get_byte_range(file, index)
            print("Byte_range = ", byte_range)

        elif command == "empty":
            byte_range = get_byte_range(file, index)
            empty_signature_content(file, byte_range)
    except Exception as e:
        print("Caught exception", e)

if __name__ == "__main__":
    import sys
    command = sys.argv[1]

    if command == "encode_oid":
        OID_encoder(sys.argv[2])
    elif command == "decode_oid":
        OID_decoder(sys.argv[2])
    elif command == "sign": 
        sign_content()
    elif command == "verify":
        verify_signature()
    else:
        process_pdf_commands(command)
```

