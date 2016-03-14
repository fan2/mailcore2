- 测试 SMTPSession

static void testSMTP(mailcore::Data * data)  
static void testSendingMessageFromFileViaSMTP(mailcore::Data * data)  
> SMTPSession(ConnectionTypeStartTLS)

- 测试 SMTPAsyncSession

static void testAsyncSMTP(mailcore::Data * data)  
static void testAsyncSendMessageFromFileViaSMTP(mailcore::Data * data)  
> SMTPAsyncSession(ConnectionTypeStartTLS) - SMTPOperation

## testAsyncSMTP
```
static void testAsyncSMTP(mailcore::Data * data)
{
    mailcore::SMTPAsyncSession * smtp;
    TestSMTPCallback * callback = new TestSMTPCallback();
    
    smtp = new mailcore::SMTPAsyncSession();
    
    smtp->setHostname(MCSTR("smtp.gmail.com"));
    smtp->setPort(25);
    smtp->setUsername(email);
    smtp->setPassword(password);
    smtp->setConnectionType(mailcore::ConnectionTypeStartTLS);
    
    mailcore::SMTPOperation * op = smtp->sendMessageOperation(data);
    op->setSmtpCallback(callback);
    op->setCallback(callback);
    op->start();
    
	mainLoop();

    //smtp->release();
}
```

### class SMTPAsyncSession

```
SMTPAsyncSession::SMTPAsyncSession()
{
    mSession = new SMTPSession();
    mQueue = new OperationQueue();
    // ...
}

SMTPSession * SMTPAsyncSession::session()
{
    return mSession;
}
```

```
SMTPOperation * SMTPAsyncSession::sendMessageOperation(Data * messageData)
{
    SMTPSendWithDataOperation * op = new SMTPSendWithDataOperation();
    op->setSession(this);
    op->setMessageData(messageData);
    return (SMTPOperation *) op->autorelease();
}
```

```
void SMTPAsyncSession::runOperation(SMTPOperation * operation)
{
    cancelDelayedPerformMethod((Object::Method) &SMTPAsyncSession::tryAutomaticDisconnectAfterDelay, NULL);
    mQueue->addOperation(operation);
}
```

### SMTPSendWithDataOperation

```
class MAILCORE_EXPORT SMTPSendWithDataOperation : public SMTPOperation
```

```
    class MAILCORE_EXPORT SMTPOperation : public Operation, public SMTPProgressCallback {
    public:
        
        virtual void setSession(SMTPAsyncSession * session);
        virtual SMTPAsyncSession * session();
        
        virtual void setSmtpCallback(SMTPOperationCallback * callback);
        virtual SMTPOperationCallback * smtpCallback();
        
        virtual void start();
    }
```

```
void SMTPOperation::start()
{
    mSession->runOperation(this);
}
```

```
void SMTPSendWithDataOperation::main()
{
    ErrorCode error;
    if (mMessageFilepath != NULL) {
        session()->session()->sendMessage(mFrom, mRecipients, mMessageFilepath, this, &error);
    }
    else
    if ((mFrom != NULL) && (mRecipients != NULL)) {
        session()->session()->sendMessage(mFrom, mRecipients, mMessageData, this, &error);
    }
    else {
        session()->session()->sendMessage(mMessageData, this, &error);
    }
    setError(error);
}
```

### SMTPSession

> SMTPSession::sendMessage
>> SMTPSession::loginIfNeeded
>>
>>> connect(mailsmtp_socket_connect/mailsmtp_ssl_connect)
>>> login(mailesmtp_auth)）
>>
>> esmtp_address_list_add
>> mailesmtp_send
