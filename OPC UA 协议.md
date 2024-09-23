# OPC UA 协议

## OPC UA 是什么

**OPC UA**的全名是**OPC Unified Architecture**（**OPC统一架构**）。是[OPC基金会](https://zh.wikipedia.org/w/index.php?title=OPC基金會&action=edit&redlink=1)应用在[自动化技术](https://zh.wikipedia.org/wiki/自動化技術)的[机器对机器](https://zh.wikipedia.org/wiki/機器對機器)[网络传输协定](https://zh.wikipedia.org/wiki/網路傳輸協定)。有以下的特点：

- 着重在资料收集以及控制为目的的通讯，用在工业设备以及系统中
- [开源标准](https://zh.wikipedia.org/wiki/开源标准)：标准可以免费取得，实作设备不需授权费，也没有其他限制
- [跨平台](https://zh.wikipedia.org/wiki/跨平台)：不限制操作系统或是编程语言
- [面向服务的体系结构](https://zh.wikipedia.org/wiki/服務導向架構)（SOA）
- 强健的[资讯安全](https://zh.wikipedia.org/wiki/資訊安全)特性
- 整合的[资讯模型](https://zh.wikipedia.org/w/index.php?title=資訊模型&action=edit&redlink=1)，是资讯整合中，基础设施的基础，制造商以及组织可以将其复杂的资料在OPC UA命名空间上建模，利用OPC UA面向服务的体系结构的优点。

## OPC UA报文格式

OPC UA通信模型中，客户端和服务器之间的交互是基于一系列的服务请求和响应。每个服务请求和响应都遵循OPC UA定义的编码规则，可以序列化为二进制流或者XML。

常见的OPC UA报文主要分为两类 OPC UA over TCP、OPC UA Secure Conversation

### OPC UA over TCP报文结构

OPC UA over TCP报文包括消息头和消息体，主要结构如下：

| **用途** | 消息头         | 消息体                                                       |
| -------- | -------------- | ------------------------------------------------------------ |
| **长度** | 8byte          | 不定                                                         |
| **描述** | 控制和描述报文 | 实际要传输的数据，其内容和结构取决于具体的OPC UA服务请求或响应 |

#### 2.1.1消息头

其中，消息头部分报文结构如下：

其中，消息头部分报文结构如下：

| **用途** | 消息类型         | 保留段                                                       | 消息大小                            |
| -------- | ---------------- | ------------------------------------------------------------ | ----------------------------------- |
| **长度** | 3byte            | 1byte                                                        | 4byte                               |
| **描述** | 用于标识报文类型 | 如果消息类型是OPC UA链接协议支持的值之一，则设置为“F”的ACSII码 | 整个消息头+消息体的长度，单位为字节 |

消息类型部分共分四类：

- HEL：表示消息体为Hello报文
- ACK：表示消息体为Acknowledge报文
- ERR：表示消息体为Error报文
- RHE：表示消息体为ReverseHello报文

#### 2.1.2 消息体

##### 2.1.2.1 Hello报文

当消息类型为HEL时，代表消息体部分为一个Hello报文，具体格式如下：

| **用途**       | **长度**     | **描述**                                                     |
| -------------- | ------------ | ------------------------------------------------------------ |
| 协议版本号     | 4byte        | 这个字段指示发送方使用的OPC UA规范的版本。接收方可以用这个信息来判断是否能够理解接收到的报文。 |
| 接收缓冲区大小 | 4byte        | 指定了接收方准备为此连接分配的最大消息大小。它用于流控制和避免接收方被过大的消息所淹没。单位为字节，该值必须大于8192。 |
| 发送缓冲区大小 | 4byte        | 发送缓冲区大小。这个字段指定了发送方准备为此连接使用的最大消息大小。这也是流控制的一部分，确保双方都能处理交换的数据。单位为字节，该值必须大于8192。 |
| 最大消息大小   | 4byte        | 这个字段指定了双方允许的最大消息体的大小。它用于防止因处理过大的单个消息而导致的性能问题。0表示客户端不限制。 |
| 最大分块数量   | 4byte        | 这个字段指定了应答报文可以被分割成的最大块数。这有助于管理大量数据的传输，确保即使是大消息也可以在双方之间有效地传输。0表示客户端不限制。 |
| 终端URL        | 最大4096byte | 客户端希望连接到的终端的URL。 如果长度超过4096 或无法识别URL所标识的资源，服务器应返回Bad_TcpEndpointUrlInvalid错误消息并关闭连接。 |

Hello报文是OPC UA TCP协议握手过程的一个重要部分，通过它，客户端和服务器可以交换基本的通信参数，为后续的更复杂交互建立基础。

##### 2.1.2.2 Acknowledge报文

| **用途**       | **长度** | **描述**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| 协议版本号     | 4byte    | 服务端支持的OPC UA协议的版本                                 |
| 接收缓冲区大小 | 4byte    | 指定了接收方准备为此连接分配的最大消息大小。它用于流控制和避免接收方被过大的消息所淹没。单位为字节，该值必须大于8192。 |
| 发送缓冲区大小 | 4byte    | 发送缓冲区大小。这个字段指定了发送方准备为此连接使用的最大消息大小。这也是流控制的一部分，确保双方都能处理交换的数据。单位为字节，该值必须大于8192。 |
| 最大消息大小   | 4byte    | 这个字段指定了双方允许的最大消息体的大小。它用于防止因处理过大的单个消息而导致的性能问题。0表示客户端不限制。 |
| 最大分块数量   | 4byte    | 这个字段指定了应答报文可以被分割成的最大块数。这有助于管理大量数据的传输，确保即使是大消息也可以在双方之间有效地传输。0表示客户端不限制。 |

Acknowledge报文提供了客户端和服务器之间通信所需的基本参数，确保双方能够有效地交换后续的OPC UA消息。客户端在收到Acknowledge报文后，会根据提供的参数调整自己的通信设置，随后双方可以开始正式的数据交换。

##### 2.1.2.3 Error报文

| **用途** | **长度**     | **描述**         |
| -------- | ------------ | ---------------- |
| 错误码   | 4byte        | 错误的数字代码。 |
| 原因     | 最大4096byte | 错误的详细描述。 |

错误码会随着版本更新而更新，这里提供一份当前版本（UA-1.05.03-2023-12-15）的错误码列表，具体可以参看[官方github](https://github.com/OPCFoundation/UA-Nodeset/blob/UA-1.05.03-2023-12-15/Schema/StatusCode.csv)

| **错误名**                                                   | **错误码** |
| ------------------------------------------------------------ | ---------- |
| Good                                                         | 0x00000000 |
| Uncertain                                                    | 0x40000000 |
| Bad                                                          | 0x80000000 |
| BadUnexpectedError                                           | 0x80010000 |
| BadInternalError                                             | 0x80020000 |
| BadOutOfMemory                                               | 0x80030000 |
| BadResourceUnavailable                                       | 0x80040000 |
| BadCommunicationError                                        | 0x80050000 |
| BadEncodingError                                             | 0x80060000 |
| BadDecodingError                                             | 0x80070000 |
| BadEncodingLimitsExceeded                                    | 0x80080000 |
| BadRequestTooLarge                                           | 0x80B80000 |
| BadResponseTooLarge                                          | 0x80B90000 |
| BadUnknownResponse                                           | 0x80090000 |
| BadTimeout                                                   | 0x800A0000 |
| BadServiceUnsupported                                        | 0x800B0000 |
| BadShutdown                                                  | 0x800C0000 |
| BadServerNotConnected                                        | 0x800D0000 |
| BadServerHalted                                              | 0x800E0000 |
| BadNothingToDo                                               | 0x800F0000 |
| BadTooManyOperations                                         | 0x80100000 |
| BadTooManyMonitoredItems                                     | 0x80DB0000 |
| BadDataTypeIdUnknown                                         | 0x80110000 |
| BadCertificateInvalid                                        | 0x80120000 |
| BadSecurityChecksFailed                                      | 0x80130000 |
| BadCertificatePolicyCheckFailed                              | 0x81140000 |
| BadCertificateTimeInvalid                                    | 0x80140000 |
| BadCertificateIssuerTimeInvalid                              | 0x80150000 |
| BadCertificateHostNameInvalid                                | 0x80160000 |
| BadCertificateUriInvalid                                     | 0x80170000 |
| BadCertificateUseNotAllowed                                  | 0x80180000 |
| BadCertificateIssuerUseNotAllowed                            | 0x80190000 |
| BadCertificateUntrusted                                      | 0x801A0000 |
| BadCertificateRevocationUnknown                              | 0x801B0000 |
| BadCertificateIssuerRevocationUnknown                        | 0x801C0000 |
| BadCertificateRevoked                                        | 0x801D0000 |
| BadCertificateIssuerRevoked                                  | 0x801E0000 |
| BadCertificateChainIncomplete                                | 0x810D0000 |
| BadUserAccessDenied                                          | 0x801F0000 |
| BadIdentityTokenInvalid                                      | 0x80200000 |
| BadIdentityTokenRejected                                     | 0x80210000 |
| BadSecureChannelIdInvalid                                    | 0x80220000 |
| BadInvalidTimestamp                                          | 0x80230000 |
| BadNonceInvalid                                              | 0x80240000 |
| BadSessionIdInvalid                                          | 0x80250000 |
| BadSessionClosed                                             | 0x80260000 |
| BadSessionNotActivated                                       | 0x80270000 |
| BadSubscriptionIdInvalid                                     | 0x80280000 |
| BadRequestHeaderInvalid                                      | 0x802A0000 |
| BadTimestampsToReturnInvalid                                 | 0x802B0000 |
| BadRequestCancelledByClient                                  | 0x802C0000 |
| BadTooManyArguments                                          | 0x80E50000 |
| BadLicenseExpired                                            | 0x810E0000 |
| BadLicenseLimitsExceeded                                     | 0x810F0000 |
| BadLicenseNotAvailable                                       | 0x81100000 |
| BadServerTooBusy                                             | 0x80EE0000 |
| GoodPasswordChangeRequired                                   | 0x00EF0000 |
| GoodSubscriptionTransferred                                  | 0x002D0000 |
| GoodCompletesAsynchronously                                  | 0x002E0000 |
| GoodOverload                                                 | 0x002F0000 |
| GoodClamped                                                  | 0x00300000 |
| BadNoCommunication                                           | 0x80310000 |
| BadWaitingForInitialData                                     | 0x80320000 |
| BadNodeIdInvalid                                             | 0x80330000 |
| BadNodeIdUnknown                                             | 0x80340000 |
| BadAttributeIdInvalid                                        | 0x80350000 |
| BadIndexRangeInvalid                                         | 0x80360000 |
| BadIndexRangeNoData                                          | 0x80370000 |
| BadIndexRangeDataMismatch                                    | 0x80EA0000 |
| BadDataEncodingInvalid                                       | 0x80380000 |
| BadDataEncodingUnsupported                                   | 0x80390000 |
| BadNotReadable                                               | 0x803A0000 |
| BadNotWritable                                               | 0x803B0000 |
| BadOutOfRange                                                | 0x803C0000 |
| BadNotSupported                                              | 0x803D0000 |
| BadNotFound                                                  | 0x803E0000 |
| BadObjectDeleted                                             | 0x803F0000 |
| BadNotImplemented                                            | 0x80400000 |
| BadMonitoringModeInvalid                                     | 0x80410000 |
| BadMonitoredItemIdInvalid                                    | 0x80420000 |
| BadMonitoredItemFilterInvalid                                | 0x80430000 |
| BadMonitoredItemFilterUnsupported                            | 0x80440000 |
| BadFilterNotAllowed                                          | 0x80450000 |
| BadStructureMissing                                          | 0x80460000 |
| BadEventFilterInvalid                                        | 0x80470000 |
| BadContentFilterInvalid                                      | 0x80480000 |
| BadFilterOperatorInvalid                                     | 0x80C10000 |
| BadFilterOperatorUnsupported                                 | 0x80C20000 |
| BadFilterOperandCountMismatch                                | 0x80C30000 |
| BadFilterOperandInvalid                                      | 0x80490000 |
| BadFilterElementInvalid                                      | 0x80C40000 |
| BadFilterLiteralInvalid                                      | 0x80C50000 |
| BadContinuationPointInvalid                                  | 0x804A0000 |
| BadNoContinuationPoints                                      | 0x804B0000 |
| BadReferenceTypeIdInvalid                                    | 0x804C0000 |
| BadBrowseDirectionInvalid                                    | 0x804D0000 |
| BadNodeNotInView                                             | 0x804E0000 |
| BadNumericOverflow                                           | 0x81120000 |
| BadLocaleNotSupported                                        | 0x80ED0000 |
| BadNoValue                                                   | 0x80F00000 |
| BadServerUriInvalid                                          | 0x804F0000 |
| BadServerNameMissing                                         | 0x80500000 |
| BadDiscoveryUrlMissing                                       | 0x80510000 |
| BadSempahoreFileMissing                                      | 0x80520000 |
| BadRequestTypeInvalid                                        | 0x80530000 |
| BadSecurityModeRejected                                      | 0x80540000 |
| BadSecurityPolicyRejected                                    | 0x80550000 |
| BadTooManySessions                                           | 0x80560000 |
| BadUserSignatureInvalid                                      | 0x80570000 |
| BadApplicationSignatureInvalid                               | 0x80580000 |
| BadNoValidCertificates                                       | 0x80590000 |
| BadIdentityChangeNotSupported                                | 0x80C60000 |
| BadRequestCancelledByRequest                                 | 0x805A0000 |
| BadParentNodeIdInvalid                                       | 0x805B0000 |
| BadReferenceNotAllowed                                       | 0x805C0000 |
| BadNodeIdRejected                                            | 0x805D0000 |
| BadNodeIdExists                                              | 0x805E0000 |
| BadNodeClassInvalid                                          | 0x805F0000 |
| BadBrowseNameInvalid                                         | 0x80600000 |
| BadBrowseNameDuplicated                                      | 0x80610000 |
| BadNodeAttributesInvalid                                     | 0x80620000 |
| BadTypeDefinitionInvalid                                     | 0x80630000 |
| BadSourceNodeIdInvalid                                       | 0x80640000 |
| BadTargetNodeIdInvalid                                       | 0x80650000 |
| BadDuplicateReferenceNotAllowed                              | 0x80660000 |
| BadInvalidSelfReference                                      | 0x80670000 |
| BadReferenceLocalOnly                                        | 0x80680000 |
| BadNoDeleteRights                                            | 0x80690000 |
| UncertainReferenceNotDeleted                                 | 0x40BC0000 |
| BadServerIndexInvalid                                        | 0x806A0000 |
| BadViewIdUnknown                                             | 0x806B0000 |
| BadViewTimestampInvalid                                      | 0x80C90000 |
| BadViewParameterMismatch                                     | 0x80CA0000 |
| BadViewVersionInvalid                                        | 0x80CB0000 |
| UncertainNotAllNodesAvailable                                | 0x40C00000 |
| GoodResultsMayBeIncomplete                                   | 0x00BA0000 |
| BadNotTypeDefinition                                         | 0x80C80000 |
| UncertainReferenceOutOfServer                                | 0x406C0000 |
| BadTooManyMatches                                            | 0x806D0000 |
| BadQueryTooComplex                                           | 0x806E0000 |
| BadNoMatch                                                   | 0x806F0000 |
| BadMaxAgeInvalid                                             | 0x80700000 |
| BadSecurityModeInsufficient                                  | 0x80E60000 |
| BadHistoryOperationInvalid                                   | 0x80710000 |
| BadHistoryOperationUnsupported                               | 0x80720000 |
| BadInvalidTimestampArgument                                  | 0x80BD0000 |
| BadWriteNotSupported                                         | 0x80730000 |
| BadTypeMismatch                                              | 0x80740000 |
| BadMethodInvalid                                             | 0x80750000 |
| BadArgumentsMissing                                          | 0x80760000 |
| BadNotExecutable                                             | 0x81110000 |
| BadTooManySubscriptions                                      | 0x80770000 |
| BadTooManyPublishRequests                                    | 0x80780000 |
| BadNoSubscription                                            | 0x80790000 |
| BadSequenceNumberUnknown                                     | 0x807A0000 |
| GoodRetransmissionQueueNotSupported                          | 0x00DF0000 |
| BadMessageNotAvailable                                       | 0x807B0000 |
| BadInsufficientClientProfile                                 | 0x807C0000 |
| BadStateNotActive                                            | 0x80BF0000 |
| BadAlreadyExists                                             | 0x81150000 |
| BadTcpServerTooBusy                                          | 0x807D0000 |
| BadTcpMessageTypeInvalid                                     | 0x807E0000 |
| BadTcpSecureChannelUnknown                                   | 0x807F0000 |
| BadTcpMessageTooLarge                                        | 0x80800000 |
| BadTcpNotEnoughResources                                     | 0x80810000 |
| BadTcpInternalError                                          | 0x80820000 |
| BadTcpEndpointUrlInvalid                                     | 0x80830000 |
| BadRequestInterrupted                                        | 0x80840000 |
| BadRequestTimeout                                            | 0x80850000 |
| BadSecureChannelClosed                                       | 0x80860000 |
| BadSecureChannelTokenUnknown                                 | 0x80870000 |
| BadSequenceNumberInvalid                                     | 0x80880000 |
| BadProtocolVersionUnsupported                                | 0x80BE0000 |
| BadConfigurationError                                        | 0x80890000 |
| BadNotConnected                                              | 0x808A0000 |
| BadDeviceFailure                                             | 0x808B0000 |
| BadSensorFailure                                             | 0x808C0000 |
| BadOutOfService                                              | 0x808D0000 |
| BadDeadbandFilterInvalid                                     | 0x808E0000 |
| UncertainNoCommunicationLastUsableValue                      | 0x408F0000 |
| UncertainLastUsableValue                                     | 0x40900000 |
| UncertainSubstituteValue                                     | 0x40910000 |
| UncertainInitialValue                                        | 0x40920000 |
| UncertainSensorNotAccurate                                   | 0x40930000 |
| UncertainEngineeringUnitsExceeded                            | 0x40940000 |
| UncertainSubNormal                                           | 0x40950000 |
| GoodLocalOverride                                            | 0x00960000 |
| GoodSubNormal                                                | 0x00EB0000 |
| BadRefreshInProgress                                         | 0x80970000 |
| BadConditionAlreadyDisabled                                  | 0x80980000 |
| BadConditionAlreadyEnabled                                   | 0x80CC0000 |
| BadConditionDisabled                                         | 0x80990000 |
| BadEventIdUnknown                                            | 0x809A0000 |
| BadEventNotAcknowledgeable                                   | 0x80BB0000 |
| BadDialogNotActive                                           | 0x80CD0000 |
| BadDialogResponseInvalid                                     | 0x80CE0000 |
| BadConditionBranchAlreadyAcked                               | 0x80CF0000 |
| BadConditionBranchAlreadyConfirmed                           | 0x80D00000 |
| BadConditionAlreadyShelved                                   | 0x80D10000 |
| BadConditionNotShelved                                       | 0x80D20000 |
| BadShelvingTimeOutOfRange                                    | 0x80D30000 |
| BadNoData                                                    | 0x809B0000 |
| BadBoundNotFound                                             | 0x80D70000 |
| BadBoundNotSupported                                         | 0x80D80000 |
| BadDataLost                                                  | 0x809D0000 |
| BadDataUnavailable                                           | 0x809E0000 |
| BadEntryExists                                               | 0x809F0000 |
| BadNoEntryExists                                             | 0x80A00000 |
| BadTimestampNotSupported                                     | 0x80A10000 |
| GoodEntryInserted                                            | 0x00A20000 |
| GoodEntryReplaced                                            | 0x00A30000 |
| UncertainDataSubNormal                                       | 0x40A40000 |
| GoodNoData                                                   | 0x00A50000 |
| GoodMoreData                                                 | 0x00A60000 |
| BadAggregateListMismatch                                     | 0x80D40000 |
| BadAggregateNotSupported                                     | 0x80D50000 |
| BadAggregateInvalidInputs                                    | 0x80D60000 |
| BadAggregateConfigurationRejected                            | 0x80DA0000 |
| GoodDataIgnored                                              | 0x00D90000 |
| BadRequestNotAllowed                                         | 0x80E40000 |
| BadRequestNotComplete                                        | 0x81130000 |
| BadTransactionPending                                        | 0x80E80000 |
| BadTicketRequired                                            | 0x811F0000 |
| BadTicketInvalid                                             | 0x81200000 |
| BadLocked                                                    | 0x80E90000 |
| BadRequiresLock                                              | 0x80EC0000 |
| GoodEdited                                                   | 0x00DC0000 |
| GoodPostActionFailed                                         | 0x00DD0000 |
| UncertainDominantValueChanged                                | 0x40DE0000 |
| GoodDependentValueChanged                                    | 0x00E00000 |
| BadDominantValueChanged                                      | 0x80E10000 |
| UncertainDependentValueChanged                               | 0x40E20000 |
| BadDependentValueChanged                                     | 0x80E30000 |
| GoodEdited_DependentValueChanged                             | 0x01160000 |
| GoodEdited_DominantValueChanged                              | 0x01170000 |
| GoodEdited_DominantValueChanged_DependentValueChanged        | 0x01180000 |
| BadEdited_OutOfRange                                         | 0x81190000 |
| BadInitialValue_OutOfRange                                   | 0x811A0000 |
| BadOutOfRange_DominantValueChanged                           | 0x811B0000 |
| BadEdited_OutOfRange_DominantValueChanged                    | 0x811C0000 |
| BadOutOfRange_DominantValueChanged_DependentValueChanged     | 0x811D0000 |
| BadEdited_OutOfRange_DominantValueChanged_DependentValueChanged | 0x811E0000 |
| GoodCommunicationEvent                                       | 0x00A70000 |
| GoodShutdownEvent                                            | 0x00A80000 |
| GoodCallAgain                                                | 0x00A90000 |
| GoodNonCriticalTimeout                                       | 0x00AA0000 |
| BadInvalidArgument                                           | 0x80AB0000 |
| BadConnectionRejected                                        | 0x80AC0000 |
| BadDisconnect                                                | 0x80AD0000 |
| BadConnectionClosed                                          | 0x80AE0000 |
| BadInvalidState                                              | 0x80AF0000 |
| BadEndOfStream                                               | 0x80B00000 |
| BadNoDataAvailable                                           | 0x80B10000 |
| BadWaitingForResponse                                        | 0x80B20000 |
| BadOperationAbandoned                                        | 0x80B30000 |
| BadExpectedStreamToBlock                                     | 0x80B40000 |
| BadWouldBlock                                                | 0x80B50000 |
| BadSyntaxError                                               | 0x80B60000 |
| BadMaxConnectionsReached                                     | 0x80B70000 |
| UncertainTransducerInManual                                  | 0x42080000 |
| UncertainSimulatedValue                                      | 0x42090000 |
| UncertainSensorCalibration                                   | 0x420A0000 |
| UncertainConfigurationError                                  | 0x420F0000 |
| GoodCascadeInitializationAcknowledged                        | 0x04010000 |
| GoodCascadeInitializationRequest                             | 0x04020000 |
| GoodCascadeNotInvited                                        | 0x04030000 |
| GoodCascadeNotSelected                                       | 0x04040000 |
| GoodFaultStateActive                                         | 0x04070000 |
| GoodInitiateFaultState                                       | 0x04080000 |
| GoodCascade                                                  | 0x04090000 |
| BadDataSetIdInvalid                                          | 0x80E70000 |

##### 2.1.2.4 ReverseHello 报文

| **用途**  | **长度**     | **描述**                                   |
| --------- | ------------ | ------------------------------------------ |
| 服务器URI | 最大4096byte | 发送消息的服务器的ApplicationUri。         |
| 终端URL   | 最大4096byte | 客户端在建立SecureChannel时使用的端点的URL |

对于基于连接的协议，如TCP，ReverseHello消息允许防火墙后面的服务器没有打开端口连接到客户端，并请求客户端使用服务器创建的套接字建立SecureChannel。

对于基于消息的协议，ReverseHello消息允许服务器向客户端通告它们的存在。在这种情况下，终端URL指定服务器的特定地址和访问它所需的任何令牌。

### OPC UA Secure Conversation报文结构

OPC UA Secure Conversation（OPC UA 安全会话）的报文格式设计用于在客户端与服务器之间建立和维护一个加密和签名的通信通道。

| **用途** | 消息头         | 安全头                                                 | 序列头             | 载荷                                                         | 安全脚                                                       |
| -------- | -------------- | ------------------------------------------------------ | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **长度** | 12byte         | 不定                                                   | 8byte              | 不定                                                         | 不定                                                         |
| **描述** | 控制和描述报文 | 包含安全相关的信息。根据对称、不对称安全算法有不同长度 | 包括序列号和请求ID | 这是实际的应用数据部分，根据安全头中定义的安全策略，它可能被加密和/或签名。 | （可选）如果报文被签名，这部分包含签名。不是所有的安全策略都需要签名。 |

#### 2.2.1 消息头

| **用途** | 消息类型         | 是否终结                                            | 消息大小                       | 安全通道ID                            |
| -------- | ---------------- | --------------------------------------------------- | ------------------------------ | ------------------------------------- |
| **长度** | 3byte            | 1byte                                               | 4byte                          | 4byte                                 |
| **描述** | 用于标识报文类型 | 一个字节的ASCII代码，指示是否是消息中的最后一个块。 | 从消息头开始的长度，单位为字节 | 服务器分配的SecureChannel的唯一标识符 |

##### 2.2.1.1 消息类型

消息类型主要有三种：

- MSG 使用与通道有关的密钥加密的消息
- OPN 打开安全通道消息
- CLO 关闭安全通道消息

##### 2.2.1.2 是否终结

一个字节的ASCII代码，指示MessageChunk是否是消息中的最后一个块。

定义为以下值：

- C 中间块。
- F 最后一个块。
- A 最后一个块（当发生错误并且消息被中止时使用）。

此字段仅对消息类型是MSG有意义，对于其他消息类型，此字段始终为“F”。

##### 2.2.1.3 消息大小

从消息头开始的长度，单位为字节

##### 2.2.1.4 安全通道ID

服务器分配的SecureChannel的唯一标识符，如果服务器接收到无法识别的安全用户ID，则应返回相应的传输层错误。

#### 2.2.2 安全头

定义了消息应用了哪些加密操作，有两个版本：非对称算法安全头和对称算法安全头。

##### 2.2.2.1 非对称算法安全头

| **用途**           | **长度** | **描述**                                                     |
| ------------------ | -------- | ------------------------------------------------------------ |
| 安全策略URI长度    | 4byte    | 安全策略URI的长度。如果未指定URI，则此值可能为0或-1。其他负值无效。单位为字节。 |
| 安全策略URI        | 不定     | 用于保护消息的安全策略的URI。此字段编码为不带空结束符的UTF-8字符串。 |
| 发送方证书长度     | 4byte    | 发送方证书的长度。如果未指定证书，则此值可能为0或-1。其他负值无效。单位为字节。 |
| 发送方证书         | 不定     | 发送方证书                                                   |
| 接收方证书指纹长度 | 4byte    | 接收方证书指纹的长度，如果已加密，则此字段的值为20。如果未加密，则值可以是0或-1。单位为字节。 |
| 接收方证书指纹     | 不定     | 接收方证书指纹。如果消息未加密，则此字段应为空。             |

##### 2.2.2.2对称算法安全头

| **用途** | **长度** | **描述**                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| 令牌ID   | 4byte    | 用于保护消息的安全通道安全令牌的唯一标识符。如果服务器接收到它无法识别的令牌ID，它将返回相应的传输层错误。 |

#### 2.2.3 序列头

| **用途** | **长度** | **描述**                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| 序列号   | 4byte    | 由发送方分配的单调递增序列号。                               |
| 请求ID   | 4byte    | 由客户端分配给OPC UA请求消息的标识符。请求和相关响应的所有消息都使用相同的标识符。 |

#### 2.2.4 载荷

这是报文的主体部分，包含了实际的操作请求或响应数据。载荷的大小是可变的，取决于实际传输的数据量。

#### 2.2.5 安全脚

这部分是可选的，仅在使用某些特定的安全策略时才存在。它包含了额外的安全信息，比如填充数据和签名。安全脚大小同样是可变的，取决于使用的安全策略和数据。

# 参考资料

1. [OPC UA 协议：特性、工作原理及其与 MQTT 的结合 | EMQ (emqx.com)](https://www.emqx.com/zh/blog/opc-ua-protocol)
2. [OPC协议解析-OPC UA OPC统一架构 - .Ding - 博客园 (cnblogs.com)](https://www.cnblogs.com/meandme/p/10069227.html)
3. [OPC UA - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/OPC_UA)
4. [OPC报文详解 - 杜衡老师 - 博客园 (cnblogs.com)](https://www.cnblogs.com/AsarumMaxim/p/18099070)

# 协议可能存在的问题

1. 消息大小与消息体不匹配：如果消息大小字段的值小于消息长度，是否会带来缓冲区溢出的问题
2. 消息大小字段、接收缓冲区/发送缓冲区大小字段可以指定高达2^32的内存分配，可能会引起资源耗尽从而导致拒绝服务
3. 终端URL是否会导致潜在的安全问题？例如连接到恶意URL
4. 