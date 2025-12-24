# Gestión de Recursos con Etiquetas en Amazon EC2 (AWS Lab)

## Descripción general
En este laboratorio se utiliza el **sistema de etiquetas (tags) de AWS** para **identificar, administrar y automatizar acciones** sobre recursos de **Amazon EC2**.

El laboratorio se divide en dos partes:
- **Task:** uso de AWS CLI para consultar y modificar recursos según sus etiquetas.
- **Challenge:** diseño de un enfoque automatizado para **terminar instancias no conformes** (sin etiquetas obligatorias).

Este escenario refleja prácticas reales de **gobernanza, control de costos y seguridad** en entornos cloud.

---

## Arquitectura
La arquitectura del laboratorio incluye:
- Amazon VPC (Lab VPC)
- Subnet pública y privada
- Instancia **CommandHost** (AWS CLI preconfigurado)
- Múltiples instancias EC2 con etiquetas personalizadas

<img width="1130" height="426" alt="image" src="https://github.com/user-attachments/assets/c311f291-f30c-4caa-a984-a8f168ada066" />

---

## Objetivos
- Aplicar y modificar etiquetas en recursos EC2
- Buscar recursos utilizando tags
- Usar AWS CLI con filtros y JMESPath
- Automatizar acciones (stop/start/terminate) basadas en etiquetas
- Implementar un enfoque *tag-or-terminate*

---

## Servicios utilizados
- Amazon EC2
- AWS CLI
- AWS SDK for PHP
- Amazon VPC

---

## Implementación del laboratorio

### 1. Acceso al Command Host
Se estableció conexión SSH con la instancia **CommandHost**, desde donde se ejecutaron todos los comandos AWS CLI.

<img width="908" height="120" alt="image" src="https://github.com/user-attachments/assets/80feae75-919a-4b28-b9aa-ef4c49db659e" />

---

### 2. Identificación de instancias mediante tags
Se usó AWS CLI para identificar instancias EC2 pertenecientes al proyecto **ERPSystem**.

```bash
aws ec2 describe-instances --filter "Name=tag:Project,Values=ERPSystem"
```
<details>
  <summary>Ver salida del comando en formato json</summary>
  ```json
  {
    "Reservations": [
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.79",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0f96c31db32a8a389",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-79.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "ea8d1015-96ca-b5e9-08c6-1644de2f9f5f",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:69:df:ca:94:55",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-0937ed3aad12537d9",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-79.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.79"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-79.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-0978a8b3c26232995",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.79"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-02e3ce3286372a748",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        },
                        {
                            "Value": "staging",
                            "Key": "Environment"
                        },
                        {
                            "Value": "Instance4",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "web server",
                            "Key": "Name"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-064080f361c3c7e82",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.203",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-034135aa1535b7024",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-203.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "7a181ccf-30f3-bd61-be19-3d05d58d0b94",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:15:a9:ec:f9:c9",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-090ae5c204e7ca038",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-203.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.203"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-203.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-09e6d98cab42bbf81",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.203"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-0640a1ec76ecf575d",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "portal",
                            "Key": "Application"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        },
                        {
                            "Value": "web server",
                            "Key": "Name"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "Instance7",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "production",
                            "Key": "Environment"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-00aeeae33f5b0eb25",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.159",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0cb534bf1ccddf551",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-159.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "c177494a4587122l13295314t1w548161972582-WideOpenSecurityGroup-ZJ7AOM3Hgnvq",
                            "GroupId": "sg-0319664375bccc723"
                        }
                    ],
                    "ClientToken": "9f420a35-b1a2-8f3e-6929-47d467fd79b9",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:33:62:47:b0:93",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-05cd9671025446e86",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-159.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.159"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-159.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-03c5bd648f0cb145a",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "c177494a4587122l13295314t1w548161972582-WideOpenSecurityGroup-ZJ7AOM3Hgnvq",
                                    "GroupId": "sg-0319664375bccc723"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.159"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-08f62ccb6c6f2ddd7",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "app server",
                            "Key": "Name"
                        },
                        {
                            "Value": "Instance2",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "development",
                            "Key": "Environment"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-095d6f8fd41b63271",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.118",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-02a90a5f3f9f52799",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-118.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "da8cd6e7-9d4f-ebcc-0f08-b8b20baebad3",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:4f:3f:18:fe:b7",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-0cbefa29e17352270",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-118.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.118"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-118.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-03ec97bc7179fa101",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.118"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-0228ae5779f0a0897",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "staging",
                            "Key": "Environment"
                        },
                        {
                            "Value": "Instance5",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "app server",
                            "Key": "Name"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-047f39b1ec48e05f1",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.31",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0e51ac308a8ce4a51",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-31.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "be2f1522-ff82-b78c-33e7-2968694673df",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:8c:fe:9f:24:37",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-0532b98cca5cb53dd",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-31.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.31"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-31.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-02405e2158a29b729",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.31"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-03252e60cef786c69",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "Instance6",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "web server",
                            "Key": "Name"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        },
                        {
                            "Value": "production",
                            "Key": "Environment"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-0f8876c29700788d4",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.187",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0f273fd2537d6fc99",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-187.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "340a8ef1-cacd-6004-c46d-e837d7e70d11",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:4a:ca:23:02:4b",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-042529e41326c4005",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-187.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.187"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-187.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-0ed9536504d2f6d35",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.187"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-0bbaf6a6285b68e63",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        },
                        {
                            "Value": "web server",
                            "Key": "Name"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "Instance1",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "development",
                            "Key": "Environment"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-0f842ea7a547474af",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        },
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "PublicDnsName": "",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "EbsOptimized": false,
                    "LaunchTime": "2025-12-24T12:39:15.000Z",
                    "PrivateIpAddress": "10.5.1.109",
                    "ProductCodes": [],
                    "VpcId": "vpc-033d241010d4e5125",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 2
                    },
                    "StateTransitionReason": "",
                    "InstanceId": "i-0276a6634f7a41546",
                    "EnaSupport": true,
                    "ImageId": "ami-022bee044edfca8f1",
                    "PrivateDnsName": "ip-10-5-1-109.us-west-2.compute.internal",
                    "KeyName": "vockey",
                    "SecurityGroups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-07fb1e67de9174fd7"
                        }
                    ],
                    "ClientToken": "a18eef71-ad2b-e67a-ecca-aa4a29cbe146",
                    "SubnetId": "subnet-0c3d8b4eb959d2172",
                    "InstanceType": "t3.micro",
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use",
                            "MacAddress": "02:08:7e:ec:0d:61",
                            "SourceDestCheck": true,
                            "VpcId": "vpc-033d241010d4e5125",
                            "Description": "",
                            "NetworkInterfaceId": "eni-06fd5832bae8af898",
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-5-1-109.us-west-2.compute.internal",
                                    "Primary": true,
                                    "PrivateIpAddress": "10.5.1.109"
                                }
                            ],
                            "PrivateDnsName": "ip-10-5-1-109.us-west-2.compute.internal",
                            "InterfaceType": "interface",
                            "Attachment": {
                                "Status": "attached",
                                "DeviceIndex": 0,
                                "DeleteOnTermination": true,
                                "AttachmentId": "eni-attach-0a1af8ebf0f6a28d5",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            },
                            "Groups": [
                                {
                                    "GroupName": "default",
                                    "GroupId": "sg-07fb1e67de9174fd7"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "OwnerId": "548161972582",
                            "SubnetId": "subnet-0c3d8b4eb959d2172",
                            "PrivateIpAddress": "10.5.1.109"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Placement": {
                        "Tenancy": "default",
                        "GroupName": "",
                        "AvailabilityZone": "us-west-2a"
                    },
                    "Hypervisor": "xen",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda",
                            "Ebs": {
                                "Status": "attached",
                                "DeleteOnTermination": true,
                                "VolumeId": "vol-0c2770bd5ac4e3e4b",
                                "AttachTime": "2025-12-24T12:39:15.000Z"
                            }
                        }
                    ],
                    "Architecture": "x86_64",
                    "RootDeviceType": "ebs",
                    "RootDeviceName": "/dev/xvda",
                    "VirtualizationType": "hvm",
                    "Tags": [
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "cloudlab"
                        },
                        {
                            "Value": "staging",
                            "Key": "Environment"
                        },
                        {
                            "Value": "ERPSystem",
                            "Key": "Project"
                        },
                        {
                            "Value": "1.0",
                            "Key": "Version"
                        },
                        {
                            "Value": "HR",
                            "Key": "Department"
                        },
                        {
                            "Value": "web server",
                            "Key": "Name"
                        },
                        {
                            "Value": "Instance3",
                            "Key": "aws:cloudformation:logical-id"
                        },
                        {
                            "Value": "arn:aws:cloudformation:us-west-2:548161972582:stack/c177494a4587122l13295314t1w548161972582/741ca9a0-e0c5-11f0-8b1a-061340c8013b",
                            "Key": "aws:cloudformation:stack-id"
                        },
                        {
                            "Value": "c177494a4587122l13295314t1w548161972582",
                            "Key": "aws:cloudformation:stack-name"
                        },
                        {
                            "Value": "portal",
                            "Key": "Application"
                        }
                    ],
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpEndpoint": "enabled",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1
                    },
                    "AmiLaunchIndex": 0
                }
            ],
            "ReservationId": "r-0124d8f28975acff5",
            "RequesterId": "658754138699",
            "Groups": [],
            "OwnerId": "548161972582"
        }
    ]
}
```
</details>

<img width="902" height="993" alt="image" src="https://github.com/user-attachments/assets/32014eb9-8b56-4941-b760-71b55d64dbb7" />

---

### 3. Uso de JMESPath para salida estructurada
Se aplicaron consultas `--query` para obtener resultados legibles y específicos.

```bash
aws ec2 describe-instances  --filter "Name=tag:Project,Values=ERPSystem"  --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'
```

<img width="1661" height="855" alt="image" src="https://github.com/user-attachments/assets/60620223-c0c8-478d-a438-b37b3da2cad3" />

---

### 4. Visualización de etiquetas personalizadas
Se incluyeron los valores de **Project**, **Environment** y **Version** en la salida.

<img width="1896" height="941" alt="image" src="https://github.com/user-attachments/assets/d020504d-3956-423c-ba97-c2f9fa176c70" />

---

### 5. Modificación masiva de tags (Version)
Se ejecutó un script Bash para actualizar la etiqueta **Version** de instancias *development*.

```bash
./change-resource-tags.sh
```

<img width="1899" height="955" alt="image" src="https://github.com/user-attachments/assets/e87e05b5-cd2e-4a3b-b12b-a4c3adf7ad3e" />

---

## Automatización basada en etiquetas

### 6. Detención de instancias por tag
Se utilizó el script **stopinator.php** para detener instancias de desarrollo.

```bash
./stopinator.php -t"Project=ERPSystem;Environment=development"
```

<img width="598" height="929" alt="image" src="https://github.com/user-attachments/assets/252b7942-a5dd-4677-9993-382717f98d23" />

<img width="859" height="406" alt="image" src="https://github.com/user-attachments/assets/b1bb309f-2893-40cd-9f8a-aa58375a417a" />

---

### 7. Inicio de instancias por tag
Las mismas instancias fueron iniciadas nuevamente.

```bash
./stopinator.php -t"Project=ERPSystem;Environment=development" -s
```

<img width="615" height="859" alt="image" src="https://github.com/user-attachments/assets/ea6c725e-546b-4775-9c20-3e47990a0415" />

<img width="1019" height="407" alt="image" src="https://github.com/user-attachments/assets/bf8b54ec-629b-4548-8708-441ca31800a4" />

---

## Challenge: Tag-Or-Terminate

### 8. Identificación de instancias no conformes
Se simularon instancias sin la etiqueta **Environment**.

<img width="1078" height="396" alt="image" src="https://github.com/user-attachments/assets/b14163e6-cb6e-48b4-9e28-650f2b92cecf" />

---

### 9. Terminación automática de instancias
Se ejecutó el script **terminate-instances.php** para eliminar instancias no conformes.

```bash
./terminate-instances.php -region <region> -subnetid <subnet-id>
```

<img width="700" height="137" alt="image" src="https://github.com/user-attachments/assets/e3eec610-1bc4-4e40-b4ce-714805d3afc2" />

<img width="1644" height="195" alt="image" src="https://github.com/user-attachments/assets/0a4c2d8b-8262-481b-8b05-15adb5533c4c" />

---

## Resultados
- Recursos administrados eficientemente mediante etiquetas
- Automatización segura de operaciones EC2
- Eliminación de recursos no conformes
- Mejora en gobernanza y control operativo

---

## Qué haría distinto en producción
- Usar **AWS Config Rules** para compliance automático
- Implementar **IAM con mínimo privilegio**
- Automatizar con **Lambda + EventBridge**
- Bloquear creación de recursos sin tags obligatorios
- Integrar con políticas de costos

---

## Conclusión
Este laboratorio demuestra cómo las **etiquetas son fundamentales para la automatización, seguridad y gobernanza** de recursos en AWS, permitiendo operaciones eficientes y escalables.
