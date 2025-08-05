# Using the Microphone capabilities in Adobe AIR 2

by Nick Kwiatkowski

![Nick Kwiatkowski](./img/nick_kwiatkowski_bio.jpg.adimg.mw.160.png)

## Requirements

### Prerequisite knowledge

This article is intended for developers who are comfortable with ActionScript 3
and have a basic understanding of ByteArrays.

### User Level

Intermediate

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)

### Sample files

- [audio_sampler project](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/audio_sampler)
  (5 KB)
  -
- [dtmf_sampler project](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/dtmf_sampler)
  (11 KB)
  -

AIR 2 introduces the ability to manipulate and record sound directly from a
microphone or the line-in on the user's sound card. Previously, the developer
had to involve the use of a remote server such as the Adobe Flash Media Server
to access any of these inputs.

AIR 2 exposes the `SampleDataEvent.SAMPLE_DATA` event for microphone or line-in
inputs, which enables you to access the raw data from the sound card as if you
were reading it from a file on the file system via the `Sound.extract()` method.
With this raw data you can read this data to add effects, or, in the case of the
example discussed here, search for certain audio patterns. Additionally, using
the file-system access that the AIR runtime gives you, you can save this raw
data to the hard drive or encapsulate it as a WAV file.

### Setting up and working with the microphone

Up to the release of Adobe AIR 2, just like in Flash 10 and below, developers
had no direct access to any of the data coming in from the user's microphone.
The only way to have access to this data was to connect the AIR runtime or Flash
Player to a remote server, such as Adobe Flash Media Server, to which the
microphone data could be sent. AIR 2 introduces the
`SampleDataEvent.SAMPLE_DATA` event so that we can get access to this raw data
to manipulate or store it. For example, you can now add simple echo effects,
audio gates, or other sound functions to enhance the sound coming from the
input.

You can attach as many microphone (or line-in) devices as you want. You can also
attach to a microphone that is already working with a
`NetStream.attachMicrophone()` output, allowing you to locally store any data
that is being sent to the server. The raw data returned is formatted as a
`ByteArray` of single-channel (mono) float value, uncompressed PCM samples.

#### Selecting the microphone input

Before you start to receive data from your microphone input, you will need to
select the input from which you want to get the data. You do this by first
importing the new `flash.media.Microphone` class into your project. Next, you
will want to display a list of available microphones to end users so they can
select the proper input. Unlike Flash Player, AIR will not display the security
dialog box where users can select the input; you must do this yourself. The list
of available microphones are available in the `Microphone `singleton, within
the` names` property. This array contains the names (as strings) of the input
sources that are returned by the operating system. You will need to know the
index number of the microphone that the user selected from this array. The
system's default microphone will have an index of 0.

    <mx:Script>
    	<![CDATA[
    		import flash.media.Microphone;
    		[Bindable] private var microphoneList:Array;
    		public function setupMicrophoneList():void
    		{
    			microphoneList = Microphone.names;
    		}
    	]]>

    </mx:Script>

    <mx:ComboBox id="comboMicList" dataProvider="{microphoneList}" />

#### Setting up the microphone instance

After you have figured out which microphone the user wants to use, you will need
to instantiate a new `Microphone` object. Set your microphone instance to a copy
of the `Microphone` singleton's `getMicrophone()` function, passing in the index
of the selected microphone (as shown in the following snippet). After a
microphone is selected and the object is instantiated, this object will spawn
`SampleDataEvent.SAMPLE_DATA` events that contain the raw PCM data. To start
receiving microphone events, you will need to attach an event handler to the
`SAMPLE_DATA` event. To stop receiving the microphone event, you need to remove
that event handler.

    <mx:Script>
    	<![CDATA[
    		import flash.events.SampleDataEvent;
    		import flash.media.Microphone;

    		[Bindable] private var microphoneList:Array;
    		protected var microphone:Microphone;
    		protected var isRecording:Boolean = false;
    		protected function setupMicrophoneList():void
    		{
    			microphoneList = Microphone.names;
    		}
    		protected function setupMicrophone():void
    		{
    			microphone = Microphone.getMicrophone(comboMicList.selectedIndex);
    		}

    		protected function startMicRecording():void
    		{
    			isRecording = true;
    			microphone.addEventListener(SampleDataEvent.SAMPLE_DATA, gotMicData);
    		}

    		protected function stopMicRecording():void
    		{
    			isRecording = false;
    			microphone.removeEventListener(SampleDataEvent.SAMPLE_DATA, gotMicData);
    		}

    		private function gotMicData(micData:SampleDataEvent):void
    		{
    			// micData.data contains a ByteArray with our sample.
    		}
    	]]>
    </mx:Script>

    <mx:ComboBox id="comboMicList" dataProvider="{microphoneList}"/>
    <mx:Button label="Start Rec" click="startMicRecording()"/>
    <mx:Button label="Stop Rec" click="stopMicRecording()"/>
    <mx:Button label="Select Mic" click="setupMicrophone()"/>

#### Doing something with the sampled data

When you have the event firing with active data, you will most likely want to do
something with it. To finish up this first example, you will simply play back
the sound that you are recording. Add the following code to your existing
project, and don't forget to add a button to call the `playbackData()` function.

    import flash.media.Sound;
    import flash.utils.ByteArray;
    protected var soundRecording:ByteArray;
    protected var soundOutput:Sound;
    protected function playbackData():void
    {
    	soundRecording.position = 0;

    	soundOutput = new Sound();
    	soundOutput.addEventListener(SampleDataEvent.SAMPLE_DATA, playSound);

    	soundOutput.play();
    }

    private function playSound(soundOutput:SampleDataEvent):void
    {
    	if (!soundRecording.bytesAvailable > 0)
    		return;
    	for (var i:int = 0; i < 8192; i++)
    	{
    		var sample:Number = 0;
    		if (soundRecording.bytesAvailable > 0)
    			sample = soundRecording.readFloat();
    		soundOutput.data.writeFloat(sample);
    		soundOutput.data.writeFloat(sample);
    	}
    }

In addition to simply playing back the audio, you can also encapsulate your raw
data into a WAV file that you can store to the use's hard drive. With the help
of the WAVWriter class included in the samples downloads, you can transcode your
recording into a specific bitrate, and encapsulate the data with the proper
headers for writing to disk. Since WAV files are just ByteArray streams of PCM
data with a header on the front, it is very easy to take your raw data and
create these files. To save WAV files in your application, import the
com.adobe.audio.format.WAVWriter class and add the following function:

    protected function saveFile():void
    {
    	var outputFile:File = File.desktopDirectory.resolvePath("recording.wav");
    	var outputStream:FileStream = new FileStream();
    	var wavWriter:WAVWriter = new WAVWriter();
    	soundRecording.position = 0;  // rewind to the beginning of the sample

    	wavWriter.numOfChannels = 1; // set the inital properties of the Wave Writer
    	wavWriter.sampleBitRate = 16;
    	wavWriter.samplingRate = 44100;

    	wavWriter.processSamples(outputStream, soundRecording, 44100, 1); // convert our ByteArray to a WAV file.
    	outputStream.open(outputFile, FileMode.WRITE);  //write out our file to disk.
    	outputStream.close();
    }

#### Setting the microphone properties

The Microphone object makes many properties available to help you control the
quality and type of data you get back from it. For example, you can set
driver-level settings such as the microphone gain, silence level, and rate. If
you are familiar with Adobe Connect, these settings are all set via the Audio
Wizard and can be fine-tuned to give the user more control of their microphone
enviroment. All of these settings can be made via the microphone instance.

You may also want to enable the user to mute the microphone. (If you do this,
your `SampleDataEvent` will simply send back `ByteArrays` full of zeros.)
Quality settings can be set via the `rate` property, which determines how many
samples the microphone wll pass back to you per second. You should not change
any of these properties after you have added your event listener, or you may
have mismatched data in your ByteArray.

#### Doing something with your data

You are not limited to what you can do with your recorded data. Many developers
will find that they will want to display a volume meter or a spectrum analyzer
of the data that is currently being recorded or displayed. You can use the
`SoundMixer.ComputeSpectrum`, for example, to compute the values needed to
display a spectrum analyzer or a sine-wave representation of the current audio.
Note, however, that the ComputeSpectrum class will only work with audio that is
currently being played to the end user via the Flash Player primary audio mixer.
Remember that if you get the float values of the stream, they represent the wave
form as integers of -1 to 1 (which can be used to graph the audio wave).

### Building the DTMF parser

Once you have your audio recorded to a ByteArray, you can take the
representation of the audio and search it for patterns. DTMF tones, commonly
found on telephones, are a combination of sine waves that are embedded into a
sound recording. DTMF tones are made of two sine waves at different frequencies
that are overlapped to make the touch-tone noise that we know.

In the following example, you will enable the default microphone and records a
bit of data to a ByteArray. You will then pass this ByteArray to a function that
will attempt to find each of the valid tones. If it finds a combination of the
two, it knows what number was pushed.

To start and stop the microphone recording, execute the following code:

    	private function startMicRecording():void
    	{
    		micRecording = new ByteArray();
    		myMicrophone = Microphone.getMicrophone();
    		myMicrophone.rate = 44;
    		myMicrophone.setLoopBack(enableLoopback);
    		myMicrophone.addEventListener(SampleDataEvent.SAMPLE_DATA, gotMicData);
    	}

    	private function stopMicRecording():void
    	{
    		myMicrophone.removeEventListener(SampleDataEvent.SAMPLE_DATA, gotMicData);
    	}

Using the `gotMicData` event, you pass the data you received from the
`SAMPLE_DATA` event to your parsing function.

    	private function gotMicData(e:SampleDataEvent):void
    	{
    		var udioChunk:ByteArray = new ByteArray();
    		audioChunk.writeBytes(e.data);
    		myDisplay.text = myDisplay.text + myDTMF.searchDTMF(audioChunk);
    	}

The `searchDTMF()` function parses the audio using what is known as the
[Goertzel alrorithm](https://en.wikipedia.org/wiki/Goertzel) to see if a certain
tone exists. You will get back each tone's strength, and rank them. The tones
with the strongest representaiton will be looked up and assigned their DTMF
values.

When you run this example, start the microphone recording. Place a phone (or
cellular telephone) next to the microphone and press a number on the keypad. The
number represenging the button you pushed should show up in the
`myDisplay TextInput`. If it does not, adjust the dB sensitivity using the
slider on the bottom of the app.

### Where to go from here

This article covered the new `SampleDataEvent.SAMPLE_DATA` functionality that
exists in AIR 2. I demonstrated how to enable the microphone, record, and
manipulate the results.

- [ActionScript 3 Reference for the Flash Platform](<https://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/media/Microphone.html#getMicrophone()>)
  on `getMicrophone()` functionality
- [Goertzel alrorithm](https://en.wikipedia.org/wiki/Goertzel)
- [audio_sampler project and source code](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/audio_sampler)
- [dtmf_sampler project and source code](https://github.com/joshtynjala/adobe-developer-connection-samples-archive/tree/main/dtmf_sampler)
