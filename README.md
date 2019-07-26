# xxx

A Flutter project to reproduce the issue of gpus_ReturnNotPermittedKillClient.

## Reproduce step

1. Run the demo on iOS real device (iPhone 6 / iPhone 6 Plus / iPhone 5)
2. Wait until the demo screen shows.
3. Press mobile phone [Home] button let demo app enter background.
4. Wait a moment until the gif image download finish.
5. Crashed.


## Crash Stack

```
libGPUSupportMercury.dylib`gpus_ReturnNotPermittedKillClient:
    0x191311efc <+0>:  orr    w8, wzr, #0x1
    0x191311f00 <+4>:  mov    w9, #-0x21530000
    0x191311f04 <+8>:  movk   w9, #0xbeef
->  0x191311f08 <+12>: str    w9, [x8]
    0x191311f0c <+16>: ret    

#0	0x0000000191311f08 in gpus_ReturnNotPermittedKillClient ()
#1	0x0000000191312ec4 in gpusSubmitDataBuffers ()
#2	0x00000001003d2624 in SkImage::MakeCrossContextFromPixmap(GrContext*, SkPixmap const&, bool, SkColorSpace*, bool) ()
#3	0x0000000100182128 in flutter::MultiFrameCodec::GetNextFrameImage(fml::WeakPtr<GrContext>) ()
#4	0x00000001001821dc in flutter::MultiFrameCodec::GetNextFrameAndInvokeCallback(std::__1::unique_ptr<tonic::DartPersistentValue, std::__1::default_delete<tonic::DartPersistentValue> >, fml::RefPtr<fml::TaskRunner>, fml::WeakPtr<GrContext>, fml::RefPtr<flutter::SkiaUnrefQueue>, unsigned long) ()
#5	0x0000000100185510 in std::__1::__function::__func<fml::internal::CopyableLambda<flutter::MultiFrameCodec::getNextFrame(_Dart_Handle*)::$_3>, std::__1::allocator<fml::internal::CopyableLambda<flutter::MultiFrameCodec::getNextFrame(_Dart_Handle*)::$_3> >, void ()>::operator()() ()
#6	0x00000001001737c4 in fml::MessageLoopImpl::FlushTasks(fml::FlushType) ()
#7	0x00000001001781bc in fml::MessageLoopDarwin::OnTimerFire(__CFRunLoopTimer*, fml::MessageLoopDarwin*) ()
#8	0x0000000182cf5794 in __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__ ()
#9	0x0000000182cf5438 in __CFRunLoopDoTimer ()
#10	0x0000000182cf2b4c in __CFRunLoopRun ()
#11	0x0000000182c1cc50 in CFRunLoopRunSpecific ()
#12	0x00000001001782e0 in fml::MessageLoopDarwin::Run() ()
#13	0x0000000100173700 in fml::MessageLoopImpl::DoRun() ()
#14	0x00000001001777f8 in void* std::__1::__thread_proxy<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, fml::Thread::Thread(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&)::$_0> >(void*) ()
#15	0x00000001829a3b28 in _pthread_body ()
#16	0x00000001829a3a8c in _pthread_start ()
#17	0x00000001829a1028 in thread_start ()
```

## Reason

The key to reproduce the issue is, enable the iOS app run on background.

So, just implement the following code in AppDelegate.

```objective-c
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    [application beginBackgroundTaskWithExpirationHandler:^ { }];
}
```

After enter background, app can continue run amount 10 minutes.

So, the image downloading task in Flutter still works.

After image downloaded, the `Codec.getNextFrame` called, crashed.

Detail descripted here [https://developer.apple.com/library/archive/qa/qa1766/_index.html](https://developer.apple.com/library/archive/qa/qa1766/_index.html).

## Resolution

Prevent `Codec.getNextFrame` call, until app become active.