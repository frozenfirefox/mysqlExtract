# mysqlExtract
提取mysql的程序

<?php
    /**************************************************************
        ##########提取update 和 insert语句程序##############
        只能用cli模式运行
        php mysql.php --flag=log --filename=dhw.sql
        接收参数两个 @flag @filename
    **************************************************************/

    /**
     * [$flag description]
     * @var [type]
     * 处理文件
     */
    class DealFile{

        private $filename   = '';
        private $flag       = '';
        /**
         * 构造函数
         * @param [type] $filename [description]
         * @param [type] $flag     [description]
         */
        function __construct($filename, $flag){
            $this->filename = $filename;
            $this->flag = $flag;
        }

        /**
         * [my_scandir description]
         * 检索文件夹
         * @param  string $dir  [description]
         * @param  string $flag [description]
         * @return [type]       [description]
         */
        protected function my_scandir($dir = ".", $flag = 'log'){
            $files = [];
            if(@$handle = opendir($dir)){
                while(($file = readdir($handle)) !== false){
                    if($file != ".." && $file != "."){
                        //排除根目录
                        if(is_dir($dir."/".$file)){
                            $files[$file] = $this->my_scandir($dir."/".$file);
                        }else{
                            if(preg_match('/\.'.$flag.'/i', $file)){
                                $files[] = $file;
                            }else{
                                //只找我们需要的文件
                            }
                        }
                    }
                }
            }

            return $files;
        }

        /**
         * [readFiles description]
         * 处理文件内容
         * @param  [type] $file [description]
         * @param  [type] $k    [description]
         * @return [type]       [description]
         */
        protected function readFiles($file, $k){
            $myfile = fopen($file, "r") or die("Unable to open file!");
            $str =  fread($myfile,filesize($file));
            $subpatterns = [];
            preg_match_all('/(update\s+.*\[|insert into\s+.*\[)/i', $str, $subpatterns);
            $new_arr = [];
            $mylog = fopen($this->filename, "a+") or die("Unable to open file!");
            foreach ($subpatterns[0] as $key => $value) {
                $new_arr[] = str_replace('[', ';', $value)."\r\n";
            }
            fwrite($mylog, implode(' ', $new_arr));
            // $myfile2 = fopen('', "r") or die("Unable to open file!");
            // file_put_contents('bak.sql', implode(' ', $new_arr).PHP_EOL, LOCK_EX);
            fclose($myfile);
            fclose($mylog);
        }

        /**
         * **
         * @return [type] [description]
         */
        public function main(){
            $re = $this->my_scandir(".", $this->flag);
            $files = array_filter($re);
            if($files){
                foreach ($files as $key => $v) {
                    echo $v."\r\n";
                    $this->readFiles($v, $key);
                }
            }else{
                print('没有处理文件!');
            }
        }

    }

    /***********************************************************/
    /**
     * [$flag description]
     * 接受参数来调取信息
     * @var [type]
     */
    $longopt = [
        'flag::',
        'filename::',
    ];
    $flag = getopt('', $longopt)?:[];
    if($flag){
        $filename = $flag['filename']?:'mysql.sql';
        $flag = $flag['flag']?:'log';
    }else{
        $filename = 'mysql.sql';
        $flag = 'log';
    }

    $dealM = new DealFile($filename, $flag);
    $dealM->main();
    /***********************************************************/
?>
