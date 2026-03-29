<h1>PCM1808 ADC and a PCM5102 DAC running on 96kHz on Raspberry Pi OS Trixie</h1>

The DTS file is for this ADC and DAC:<br>
ADC: https://nl.aliexpress.com/item/32777376004.html<br>
DAC: https://nl.aliexpress.com/item/1005006323878361.html

An external oscillator is necessary to get this combination to work:<br>
https://mou.sr/4bWy2iM

hook up the ADC and DAC like this:<br>
<b>
<ul>
<li>SCK of both to the output of the 24.576MHz oscillator</li>
<li>BCK of both to RPi GPIO 18</li>
<li>LRCK of both to RPi GPIO 19</li>  
<li>OUT of PCM1808 to RPi GPIO 20</li>
<li>DIN of PCM5102 to RPi GPIO 21</li>
</ul>
</b>

Solder a wire from GND to FMY of the PCM1808 to select I2S mode. Solder MD0 and MD1 of the PCM1808 to 3.3V to select 96kHz MASTER mode using the 24.576MHz oscillator (256 x Fs). The PCM5102 autodetects the clocksignal and selects its SLAVE mode automatically. Solder the H1L, H2L, H3l and H4L pads on the PCM5102 according to the following [image](https://user-images.githubusercontent.com/77981303/232213355-93542f42-e6ee-4e16-add7-15a7b20fc30b.jpeg) to select I2S.


compile pcm5108.dts with:<br>
<b> dtc -@ -I dts -O dtb -o pcm5108.dtbo pcm5108.dts</b>

copy it to the overlay folder for Trixie:<br>
<b> sudo cp pcm5108.dtbo /boot/firmware/overlays/</b>

edit /boot/firmware/config.txt:<br>
<b>sudo nano /boot/firmware/config.txt</b>

remove or disable all other sound overlays and comment out dtparam=audio=on:<br>
<b>#dtparam=audio=on<br>
dtoverlay=vc4-kms-v3d,noaudio</b>

now add:<br>
<b>dtparam=i2s=on<br>
dtoverlay=pcm5108</b>

save the file and reboot:<br>
<b> sudo reboot now</b>

does it appear?:<br>
<b> arecord -l</b>

something like the following should show:<br>
<b>card 0: pcm5108 [pcm5108], device 0: bcm2835-i2s-dir-hifi dir-hifi-0 [bcm2835-i2s-dir-hifi dir-hifi-0]<br>
  Subdevices: 1/1<br>
  Subdevice #0: subdevice #0</b>

if yes then run to test:<br>
<b>arecord -D hw:0,0 -f S32_LE -r 96000 -c 2 -V stereo test.wav</b>

to loop the input to the output do:<br>
<b>alsaloop -C hw:0,0 -P hw:0,1 -r 96000 -f S32_LE</b>

Have fun!
