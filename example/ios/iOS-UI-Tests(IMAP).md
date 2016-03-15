## MasterViewController

```
@interface MasterViewController ()

@property (nonatomic, strong) NSArray *messages;

@property (nonatomic, strong) MCOIMAPOperation *imapCheckOp;
@property (nonatomic, strong) MCOIMAPSession *imapSession;
@property (nonatomic, strong) MCOIMAPFetchMessagesOperation *imapMessagesFetchOp;

@end
```

- MCOIMAPSession *imapSession;   // 包含 imapCheckOp
- MCOIMAPOperation *imapCheckOp; // IMAPCheckAccountOperation
- MCOIMAPFetchMessagesOperation *imapMessagesFetchOp;

## MCOIMAPCheckAccountOperation

### interface hierarchy
// mailcore2/src/objc/imap/MCOIMAPCheckAccountOperation.h
MCOIMAPCheckAccountOperation : MCOIMAPOperation : MCOIMAPBaseOperation : MCOOperation

### callee hierarchy
> -[MasterViewController viewDidLoad]
>> -[MasterViewController startLogin]
>>> -[MasterViewController loadAccountWithUsername:password:hostname:oauth2Token:]
>>>> -[MCOIMAPCheckAccountOperation start:]
>>>>> -[MCOOperation(Private) start]
>>>>>> IMAPOperation::start()

### loadAccount

```
- (void)loadAccountWithUsername:(NSString *)username
                       password:(NSString *)password
                       hostname:(NSString *)hostname
                    oauth2Token:(NSString *)oauth2Token
{
	self.imapSession = [[MCOIMAPSession alloc] init];

	self.imapCheckOp = [self.imapSession checkAccountOperation];

	[self.imapCheckOp start:^(NSError *error) {

		if (error == nil) {
			[strongSelf loadLastNMessages:NUMBER_OF_MESSAGES_TO_LOAD];
		} else {
			NSLog(@"error loading account: %@", error);
		}
		
		strongSelf.imapCheckOp = nil;
	}];

}
```

### MCOIMAPCheckAccountOperation
#### MCOIMAPCheckAccountOperation(IMAPOperation) & IMAPAsyncConnection

- **MCOIMAPCheckAccountOperation**

```
+ (void) load
{
    MCORegisterClass(self, &typeid(nativeType));
}

+ (NSObject *) mco_objectWithMCObject:(mailcore::Object *)object
{
    nativeType * op = (nativeType *) object;
    return [[[self alloc] initWithMCOperation:op] autorelease];
}
```

- MCOOperation

```
@implementation MCOOperation {
    Operation * _operation;
    MCOOperationCompletionCallback * _callback;
    BOOL _started;
}
```

- MCOOperation::start

```
- (void) MCOOperation::start
{
    _started = YES;
    [self retain];
    _operation->start();
}
```

```
void IMAPOperation::start()
{
    if (session() == NULL) {
        IMAPAsyncConnection * connection = mMainSession->sessionForFolder(mFolder, mUrgent);
        setSession(connection);
    }
    session()->runOperation(this);
}
```

```
void IMAPAsyncConnection::runOperation(IMAPOperation * operation)
{
    mQueue->addOperation(operation);
}
```

```
void IMAPCheckAccountOperation::main()
{
    ErrorCode error;
    session()->session()->connectIfNeeded(&error);
    if (error == ErrorNone) {
        session()->session()->login(&error);
        if (error != ErrorNone) {
            MC_SAFE_REPLACE_COPY(String, mLoginResponse, session()->session()->loginResponse());
        }
    }
    setError(error);
}
```

```
void OperationQueue::runOperations()
{
	op->main();
}
```

#### IMAPSession* IMAPAsyncConnection::mSession

```

IMAPAsyncConnection * IMAPOperation::session()
{
    return mSession;
}

IMAPSession * IMAPAsyncConnection::session()
{
    return mSession;
}

IMAPAsyncConnection::IMAPAsyncConnection()
{
    mSession = new IMAPSession();
    mQueue = new OperationQueue();
}
```

## MCOIMAPFetchMessagesOperation

`- [MasterViewController loadAccountWithUsername:] ` 登录成功时，调用 `-[MasterViewController loadLastNMessages:]` 拉取最近邮件列表。

1. self.imapSession fetchMessagesByNumberOperationWithFolder 拉取 inbox 文件夹，返回 `IMAPFetchMessagesOperation*`  
2. 启动 `self.imapMessagesFetchOp start -> main` 拉取邮件列表。

### interface hierarchy
- cpp

// mailcore2/src/async/imap/MCIMAPFetchMessagesOperation.h

```
#ifdef __cplusplus
IMAPFetchMessagesOperation : IMAPOperation
#endif
```

- oc

// mailcore2/src/objc/imap/MCOIMAPFetchMessagesOperation.h
MCOIMAPFetchMessagesOperation : MCOIMAPBaseOperation : MCOOperation

### loadLastNMessages

```
- (void)loadLastNMessages:(NSUInteger)nMessages
{
	NSString *inboxFolder = @"INBOX";
	MCOIMAPFolderInfoOperation *inboxFolderInfo = [self.imapSession folderInfoOperation:inboxFolder];
	
	[inboxFolderInfo start:^(NSError *error, MCOIMAPFolderInfo *info)
	{

		self.imapMessagesFetchOp =
		[self.imapSession fetchMessagesByNumberOperationWithFolder:inboxFolder
													   requestKind:requestKind
														   numbers:
		 [MCOIndexSet indexSetWithRange:fetchRange]];
		
		[self.imapMessagesFetchOp setProgress:^(unsigned int progress) {
			NSLog(@"Progress: %u of %u", progress, numberOfMessagesToLoad);
		}];
		
		__weak MasterViewController *weakSelf = self;
		[self.imapMessagesFetchOp start:
		 ^(NSError *error, NSArray *messages, MCOIndexSet *vanishedMessages)
		{
		}
	}
}
```

### main

```
void IMAPFetchMessagesOperation::main()
{

	session()->session()->syncMessagesByUIDWithExtraHeaders
	session()->session()->fetchMessagesByUIDWithExtraHeaders
	session()->session()->fetchMessagesByNumberWithExtraHeaders
	
}
```

### IMAPAsyncSession* MCOIMAPSession::_session

```
- (void) setSession:(MCOIMAPSession *)session
{
    [_session release];
    _session = [session retain];
}

- (MCOIMAPSession *) session
{
    return _session;
}
```

```
@implementation MCOIMAPBaseOperation {
    MCOIMAPBaseOperationIMAPCallback * _imapCallback;
    MCOIMAPSession * _session;
}

```

```
@implementation MCOIMAPSession {
    IMAPAsyncSession * _session;
    MCOConnectionLogger _connectionLogger;
    MCOIMAPCallbackBridge * _callbackBridge;
    MCOOperationQueueRunningChangeBlock _operationQueueRunningChangeBlock;
}

```

这里的 session()->session() 还是之前的 IMAPSession* IMAPAsyncConnection::mSession，随 IMAPAsyncConnection 构造创建。

