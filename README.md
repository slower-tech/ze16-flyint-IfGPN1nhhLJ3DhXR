
​《FFmpeg开发实战：从零基础到短视频上线》一书的“5\.1\.2  把音频流保存为PCM文件”介绍了如何把媒体文件中的音频流转存为原始的PCM音频，在样例代码的转存过程中，解码后的PCM数据未经任何加工处理，就直接保存到二进制文件。也就是说，原音频的采样频率是多少，PCM文件的采样频率也是多少；原音频的声道数量是多少，PCM文件的声道数量也是多少；原音频的采样位数是多少，PCM文件的采样位数也是多少。
 原汁原味保存的PCM文件本来也没什么问题，可是在实际应用中，有的业务场景需要特定规格的PCM音频。比如某厂家的语音识别引擎，要求只能输入16位的PCM数据，然而标准的MP3音频都采用32位采样，如此一来，得想办法把32位的MP3音频转换为16位的PCM音频才行。
考虑到使用FFmpeg的命令行转换比较方便，于是在控制台执行下面的ffmpeg格式转换指令，在转换采样频率和声道数量的同时一起转换采样位数。




```
ffmpeg -i night.mp3 -ar 16000 -ac 1 -acodec pcm_s16le night.pcm
```


谁知控制台输出以下的报错信息“pcm\_s16le codec not supported”，意思是不支持16位的PCM编码器。




```
pcm_s16le codec not supported
```


咦，FFmpeg怎么会不支持这么基本的PCM编码器呢？继续执行下面的编码器查看命令：




```
ffmpeg -encoders | grep pcm
```


发现输出的查询结果赫然出现下面的pcm\_s16le信息，说明FFmpeg默认已经支持该编码器。




```
A....D pcm_s16le            PCM signed 16-bit little-endian
```


那么为啥ffmpeg命令行无法正常转换PCM音频的采样位数呢？
搜了一圈发现没有使用ffmpeg成功转换采样位数的案例，只好先把原音频转换为32位采样的PCM文件，转换命令如下所示：




```
ffmpeg -i night.mp3 -ar 16000 -ac 1 -acodec pcm_f32le -f f32le night.pcm
```


接下来另外编写转换音频采样位数的代码convertpcm.c，代码内容如下所示：




```
#include 
#include 
#include 

int pcm32_to_pcm16(const char *filename)
{  
    FILE *fp =  fopen(filename, "rb");
    FILE *fp1 = fopen("output_16.pcm", "wb");
    unsigned char *sample = (unsigned char*)calloc(1, 4+1);
    while(!feof(fp))
    {
        fread(sample, 4, 1, fp);
        sample[4] = '\0';
        float *sample32 = (float*)sample;
        short sample16 = (short)floor( (*sample32) * 32767 );
        fwrite(&sample16, 2, 1, fp1);
    }
    free(sample);
    fclose(fp);
    fclose(fp1);
    return 0;  
}

int main(int argc, char **argv) {
    const char *src_name = "night.pcm";
    if (argc > 1) {
        src_name = argv[1];
    }
    pcm32_to_pcm16(src_name);
}
```


保存代码，然后执行下面的编译命令。




```
gcc convertpcm.c -o convertpcm 
```


编译完成，再执行下面的采样位数转换命令。




```
./convertpcm night.pcm
```


现在生成的output\_16\.pcm就是16位采样的PCM文件，可以用作语音识别了。


更多详细的FFmpeg开发知识参见[《FFmpeg开发实战：从零基础到短视频上线》](https://item.jd.com/14020415.html "《FFmpeg开发实战：从零基础到短视频上线》")一书。


 本博客参考[飞数机场](https://ze16.com)。转载请注明出处！
