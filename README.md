# cognitive-services-speech-sdk-rs

---
[![made-with-Rust](https://img.shields.io/badge/Made%20with-Rust-1f425f.svg)](https://www.rust-lang.org/)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](../../graphs/commit-activity)
[![License](https://img.shields.io/badge/License-Apache-blue.svg)](LICENSE-APACHE)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE-MIT)
[![Build Status](https://www.travis-ci.com/jabber-tools/cognitive-services-speech-sdk-rs.svg?branch=main)](https://www.travis-ci.com/github/jabber-tools/cognitive-services-speech-sdk-rs)

Pure Rust implementation of Microsoft Cognitive Speech Services SDK. Provides thin abstraction around native C API. Heavily inspired by official [Go library](https://github.com/microsoft/cognitive-services-speech-sdk-go). Provides speech-to-text, text-to-speech and bot framework dialog management capabilities. 

Pull requests welcome!

### Speech to text 
```
use cognitive_services_speech_sdk_rs as msspeech;
use log::*;
use std::env;

async fn speech_to_text() {
    let filename = env::var("WAVFILENAME").unwrap();
    let audio_config = msspeech::audio::AudioConfig::from_wav_file_input(&filename).unwrap();

    // before running this text export the below listed variables. Example:
    // export MSSubscriptionKey=32...
    // export MSServiceRegion=westeurope
    // export WAVFILENAME=/tmp/hello_rust.wav
    // cargo test
    let speech_config = msspeech::speech::SpeechConfig::from_subscription(
        env::var("MSSubscriptionKey").unwrap(),
        env::var("MSServiceRegion").unwrap(),
    )
    .unwrap();
    let mut speech_recognizer =
        msspeech::speech::SpeechRecognizer::from_config(speech_config, audio_config).unwrap();

    speech_recognizer
        .set_session_started_cb(|event| info!("set_session_started_cb {:?}", event))
        .unwrap();

    speech_recognizer
        .set_session_stopped_cb(|event| info!("set_session_stopped_cb {:?}", event))
        .unwrap();

    speech_recognizer
        .set_speech_start_detected_cb(|event| info!("set_speech_start_detected_cb {:?}", event))
        .unwrap();

    speech_recognizer
        .set_speech_end_detected_cb(|event| info!("set_speech_end_detected_cb {:?}", event))
        .unwrap();

    speech_recognizer
        .set_recognizing_cb(|event| info!("set_recognizing_cb {:?}", event.result.text))
        .unwrap();

    speech_recognizer
        .set_recognized_cb(|event| info!("set_recognized_cb {:?}", event))
        .unwrap();

    speech_recognizer
        .set_canceled_cb(|event| info!("set_canceled_cb {:?}", event))
        .unwrap();

    let result = speech_recognizer.recognize_once_async().await.unwrap();
    info!("got recognition {:?}", result);
    // actual result might differ as the speech api evolves :)
    // assert_eq!(result.text, "Hello rust.");
}
```

### Text to speech
```
use cognitive_services_speech_sdk_rs as msspeech;
use log::*;
use std::env;

async fn text_to_speech() {
    let pull_stream = msspeech::audio::PullAudioOutputStream::create_pull_stream().unwrap();
    let audio_config = msspeech::audio::AudioConfig::from_stream_output(&pull_stream).unwrap();

    // before running this text export the below listed variables. Example:
    // export MSSubscriptionKey=32...
    // export MSServiceRegion=westeurope
    // cargo test
    let speech_config = msspeech::speech::SpeechConfig::from_subscription(
        env::var("MSSubscriptionKey").unwrap(),
        env::var("MSServiceRegion").unwrap(),
    )
    .unwrap();
    let mut speech_synthesizer =
        msspeech::speech::SpeechSynthesizer::from_config(speech_config, audio_config).unwrap();

    speech_synthesizer
        .set_synthesizer_started_cb(|event| info!("synthesizer_started_cb {:?}", event))
        .unwrap();

    speech_synthesizer
        .set_synthesizer_synthesizing_cb(|event| info!("synthesizer_synthesizing_cb {:?}", event))
        .unwrap();

    speech_synthesizer
        .set_synthesizer_completed_cb(|event| info!("synthesizer_completed_cb {:?}", event))
        .unwrap();

    speech_synthesizer
        .set_synthesizer_canceled_cb(|event| info!("synthesizer_canceled_cb {:?}", event))
        .unwrap();

    match speech_synthesizer.speak_text_async("Hello Rust!").await {
        Err(err) => error!("speak_text_async error {:?}", err),
        Ok(speech_audio_bytes) => {
            info!("speech_audio_bytes {:?}", speech_audio_bytes);
        }
    }
}
```

For more see github integration tests (tests folder) and samples (examples folder).

## Build prerequisites

Currently only build on Linux is supported. Uses Clang and Microsoft Speech SDK shared libraries. Details can be found here [here](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnet%2Cwindows%2Cjre%2Cbrowser&pivots=programming-language-go)

Install following prerequisites before running *cargo build*:

```
sudo apt-get update 
sudo apt-get install clang build-essential libssl1.0.0 libasound2 wget
```