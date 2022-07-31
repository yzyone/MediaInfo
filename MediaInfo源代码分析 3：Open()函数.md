# MediaInfo源代码分析 3：Open()函数

我们来看一下MediaInfo中的Open()函数的内部调用过程

首先open函数封装了MediaInfo_Internal类中的open()函数

```
//打开文件
size_t MediaInfo::Open(const String &File_Name_)
{
    //封装了一层
    return Internal->Open(File_Name_);
}
```

查看一下MediaInfo_Internal类中的open()函数的源代码：

```
//打开文件
size_t MediaInfo_Internal::Open(const String &File_Name_)
{
    Close();
    CS.Enter();
    MEDIAINFO_DEBUG_CONFIG_TEXT(Debug+=__T("Open, File=");Debug+=Ztring(File_Name_).c_str();)
    //清空文件列表
        Config.File_Names.clear();
    if (Config.File_FileNameFormat_Get()==__T("CSV"))
    {
        Config.File_Names.Separator_Set(0, __T(","));
        Config.File_Names.Write(File_Name_);
    }
    //添加文件名到文件名列表
    else if (!File_Name_.empty())
        Config.File_Names.push_back(File_Name_);
    if (Config.File_Names.empty())
    {
        CS.Leave();
        return 0;
    }
    Config.File_Names_Pos=1;
    CS.Leave();
    //Parsing
    //解析
    if (BlockMethod==1)
    {
        if (!IsInThread) //If already created, the routine will read the new files
        {
            Run();
            IsInThread=true;
        }
        return 0;
    }
    else
    {
        Entry(); //Normal parsing
        return Count_Get(Stream_General);
    }
}
```

在打开之前首先调用Close()进行清理，将文件路径添加到一个列表（实际上是一个Vector）中，调用Entry()进行解析。

查看一下Entry()函数的源代码：

```
//解析函数
void MediaInfo_Internal::Entry()
{
    CS.Enter();
    MEDIAINFO_DEBUG_CONFIG_TEXT(Debug+=__T("Entry");)
    Config.State_Set(0);
    CS.Leave();
    //解析文件名（是否包含MMS？）
    //需要libmms
    if ((Config.File_Names[0].size()>=6
        && Config.File_Names[0][0]==__T('m')
        && Config.File_Names[0][1]==__T('m')
        && Config.File_Names[0][2]==__T('s')
        && Config.File_Names[0][3]==__T(':')
        && Config.File_Names[0][4]==__T('/')
        && Config.File_Names[0][5]==__T('/'))
        || (Config.File_Names[0].size()>=7
        && Config.File_Names[0][0]==__T('m')
        && Config.File_Names[0][1]==__T('m')
        && Config.File_Names[0][2]==__T('s')
        && Config.File_Names[0][3]==__T('h')
        && Config.File_Names[0][4]==__T(':')
        && Config.File_Names[0][5]==__T('/')
        && Config.File_Names[0][6]==__T('/')))
        #if defined(MEDIAINFO_LIBMMS_YES)
            Reader_libmms().Format_Test(this, Config.File_Names[0]);
        #else //MEDIAINFO_LIBMMS_YES
            {
            #if MEDIAINFO_EVENTS
                struct MediaInfo_Event_Log_0 Event;
                Event.EventCode=MediaInfo_EventCode_Create(MediaInfo_Parser_None, MediaInfo_Event_Log, 0);
                Event.Type=0xC0;
                Event.Severity=0xFF;
                Event.MessageCode=0;
                Event.MessageStringU=L"Libmms cupport is disabled due to compilation options";
                Event.MessageStringA="Libmms cupport is disabled due to compilation options";
                MediaInfoLib::Config.Event_Send((const int8u*)&Event, sizeof(MediaInfo_Event_Log_0));
            #endif //MEDIAINFO_EVENTS
            }
        #endif //MEDIAINFO_LIBMMS_YES
    //是否包含网络协议？（标记为://）
    //需要使用libcurl - the multiprotocol file transfer library
    else if (Config.File_Names[0].find(__T("://"))!=string::npos)
        #if defined(MEDIAINFO_LIBCURL_YES)
        {
            CS.Enter();
            if (Reader)
            {
                CS.Leave();
                return; //There is a problem
            }
            Reader=new Reader_libcurl();
            CS.Leave();
            Reader->Format_Test(this, Config.File_Names[0]);
            #if MEDIAINFO_NEXTPACKET
                if (Config.NextPacket_Get())
                    return;
            #endif //MEDIAINFO_NEXTPACKET
        }
        #else //MEDIAINFO_LIBCURL_YES
            {
            #if MEDIAINFO_EVENTS
                struct MediaInfo_Event_Log_0 Event;
                Event.EventCode=MediaInfo_EventCode_Create(MediaInfo_Parser_None, MediaInfo_Event_Log, 0);
                Event.Type=0xC0;
                Event.Severity=0xFF;
                Event.MessageCode=0;
                Event.MessageStringU=L"Libcurl support is disabled due to compilation options";
                Event.MessageStringA="Libcurl support is disabled due to compilation options";
                MediaInfoLib::Config.Event_Send((const int8u*)&Event, sizeof(MediaInfo_Event_Log_0));
            #endif //MEDIAINFO_EVENTS
            }
        #endif //MEDIAINFO_LIBCURL_YES
    #if defined(MEDIAINFO_DIRECTORY_YES)
        else if (Dir::Exists(Config.File_Names[0]))
            Reader_Directory().Format_Test(this, Config.File_Names[0]);
    #endif //MEDIAINFO_DIRECTORY_YES
    //是普通文件，并且文件存在
    #if defined(MEDIAINFO_FILE_YES)
        else if (File::Exists(Config.File_Names[0]))
        {
            #if defined(MEDIAINFO_REFERENCES_YES)
                string Dxw;
                if (Config.File_CheckSideCarFiles_Get() && !Config.File_IsReferenced_Get())
                {
                    //Zenlib中竟然包含专用于文件名操作的类！
                    FileName Test(Config.File_Names[0]);
                    //获得小写的文件名后缀
                    Ztring FileExtension=Test.Extension_Get();
                    FileExtension.MakeLowerCase();
                    //逐个比较（这些都是文本文件的后缀）
                    if (FileExtension!=__T("dfxp"))
                    {
                        Test.Extension_Set(__T("dfxp"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".dfxp" />
";
                    }
                    if (FileExtension!=__T("sami"))
                    {
                        Test.Extension_Set(__T("sami"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".sami" />
";
                    }
                    if (FileExtension!=__T("sc2"))
                    {
                        Test.Extension_Set(__T("sc2"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".sc2" />
";
                    }
                    if (FileExtension!=__T("scc"))
                    {
                        Test.Extension_Set(__T("scc"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".scc" />
";
                    }
                    if (FileExtension!=__T("smi"))
                    {
                        Test.Extension_Set(__T("smi"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".smi" />
";
                    }
                    //字幕
                    if (FileExtension!=__T("srt"))
                    {
                        Test.Extension_Set(__T("srt"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".srt" />
";
                    }
                    if (FileExtension!=__T("stl"))
                    {
                        Test.Extension_Set(__T("stl"));
                        if (File::Exists(Test))
                            Dxw+=" <clip file=""+Test.Name_Get().To_UTF8()+".stl" />
";
                    }
                    Ztring Name=Test.Name_Get();
                    Ztring BaseName=Name.SubString(Ztring(), __T("_"));
                    if (!BaseName.empty())
                    {
                        ZtringList List;
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_audio.mp4"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.dfxp"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.sami"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.sc2"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.scc"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.smi"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.srt"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_sub.stl"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.dfxp"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.sami"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.sc2"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.scc"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.smi"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.srt"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_forcesub.stl"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.dfxp"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.sami"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.sc2"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.scc"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.smi"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.srt"), Dir::Include_Files);
                        List+=Dir::GetAllFileNames(Test.Path_Get()+PathSeparator+BaseName+__T("_*_cc.stl"), Dir::Include_Files);
                        for (size_t Pos=0; Pos<List.size(); Pos++)
                            Dxw+=" <clip file=""+List[Pos].To_UTF8()+"" />
";
                    }
                    if (!Dxw.empty())
                    {
                        Dxw.insert(0, "<?xml version="1.0" encoding="UTF-8" ?>
"
                                "<indexFile xmlns="urn:digimetrics-xml-wrapper" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:digimetrics-xml-wrapper DMSCLIP.XSD">
"
                                " <clip source="main" file=""+FileName(Config.File_Names[0]).Name_Get().To_UTF8()+"."+FileName(Config.File_Names[0]).Extension_Get().To_UTF8()+"" />
");
                        Dxw.append("</indexFile>
");
                        Config.File_FileNameFormat_Set(__T("Dxw"));
                    }
                }
                //为空
                if (Dxw.empty())
                {
                    CS.Enter();
                    if (Reader)
                    {
                        CS.Leave();
                        return; //There is a problem
                    }
                    Reader=new Reader_File();
                    CS.Leave();
                    //测试格式
                    Reader->Format_Test(this, Config.File_Names[0]);
                }
                else
                {
                    //初始化Buffer
                    Open_Buffer_Init(Dxw.size(), FileName(Config.File_Names[0]).Path_Get()+PathSeparator+FileName(Config.File_Names[0]).Name_Get());
                    Open_Buffer_Continue((const int8u*)Dxw.c_str(), Dxw.size());
                    Open_Buffer_Finalize();
                }
            #else //defined(MEDIAINFO_REFERENCES_YES)
                CS.Enter();
                if (Reader)
                {
                    CS.Leave();
                    return; //There is a problem
                }
                Reader=new Reader_File();
                CS.Leave();
                Reader->Format_Test(this, Config.File_Names[0]);
            #endif //defined(MEDIAINFO_REFERENCES_YES)
            #if MEDIAINFO_NEXTPACKET
                if (Config.NextPacket_Get())
                    return;
            #endif //MEDIAINFO_NEXTPACKET
        }
    #endif //MEDIAINFO_FILE_YES
    CS.Enter();
    Config.State_Set(1);
    CS.Leave();
}
```

我们可以看出，MediaInfo支持流媒体协议作为输入源（例如mms），但前提是要使用相应的库（libmms，或者libcurl）。它会首先查找文件路径中有没有“://”，判断是否是流媒体输入。

大多数情况下，我们使用本地文件路径作为输入源。这是我们就进入了代码中“#if defined(MEDIAINFO_FILE_YES) "部分下面的代码。具体每块代码的左右都写在代码注释中了，不再细说。最后使用 Reader_File的Format_Test()测试输入文件的格式。

看一下Reader_File的Format_Test()的源代码：

```
//用于测试文件格式
size_t Reader_File::Format_Test(MediaInfo_Internal* MI, String File_Name)
{
    //std::cout<<Ztring(File_Name).To_Local().c_str()<<std::endl;

    //With Parser MultipleParsing
    /*
    MI->Open_Buffer_Init((int64u)-1, File_Name);
    if (Format_Test_PerParser(MI, File_Name))
         return 1;
    return 0; //There is a problem
    */
    //Get the Extension
    //文件后缀
    Ztring Extension=FileName::Extension_Get(File_Name);
    Extension.MakeLowerCase();
    //Search the theorical format from extension
    InfoMap &FormatList=MediaInfoLib::Config.Format_Get();
    InfoMap::iterator Format=FormatList.end();
    if (!MI->Config.File_ForceParser_Get().empty())
        Format=FormatList.find(MI->Config.File_ForceParser_Get());
    if (Format==FormatList.end())
    {
        Format=FormatList.begin();
        while (Format!=FormatList.end())
        {
            const Ztring &Extensions=FormatList.Get(Format->first, InfoFormat_Extensions);
            if (Extensions.find(Extension)!=Error)
            {
                if(Extension.size()==Extensions.size())
                    break; //Only one extenion in the list
                if(Extensions.find(Extension+__T(" "))!=Error
                || Extensions.find(__T(" ")+Extension)!=Error)
                    break;
            }
            ++Format;
        }
    }
    if (Format!=FormatList.end())
    {
        const Ztring &Parser=Format->second(InfoFormat_Parser);
        if (MI->SelectFromExtension(Parser))
        {
            //Test the theorical format
            if (Format_Test_PerParser(MI, File_Name)>0)
                 return 1;
        }
    }
    //遍历所有支持格式，测试格式
    size_t ToReturn=MI->ListFormats(File_Name);
    return ToReturn;

}
```

可以看出，最后遍历了所有支持的格式，试图找出合适的格式。

至此Open()函数的分析完毕。
