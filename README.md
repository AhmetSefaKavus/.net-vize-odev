using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace VizeOdevi
{
    class Program
    {
        static void Main(string[] args)
        {
            using (Process p = new Process())
            {
                ProcessStartInfo ps = new ProcessStartInfo();
                ps.Arguments = "-a -n";
                ps.FileName = "netstat.exe";
                ps.UseShellExecute = false;
                ps.WindowStyle = ProcessWindowStyle.Hidden;
                ps.RedirectStandardInput = true;
                ps.RedirectStandardOutput = true;
                ps.RedirectStandardError = true;

                p.StartInfo = ps;
                p.Start();

                StreamReader stdOutput = p.StandardOutput;
                StreamReader stdError = p.StandardError;

                string content = stdOutput.ReadToEnd() + stdError.ReadToEnd();
                string exitStatus = p.ExitCode.ToString();

                if (exitStatus != "0")
                {
                    // Command Errored. Handle Here If Need Be
                }


                string[] rows = Regex.Split(content, "\r\n");


                foreach (var item in rows)
                {

                    if (item == "") continue;
                    if (item.Trim() == "Active Connections") continue;
                    if (item == "UDP") continue;
                    if (item.Contains("LISTENING")) continue;
                    if (item.Contains("127.0.0.1")) continue;
                    if (item.Contains("%")) continue;
                    if (item.Contains("TCP"))
                    {
                        string[] split = Regex.Split(item, "\\s+");
                        Json_veri port = new Json_veri();
                        port.foreignAddress = split[3];

                        string[] split2 = item.Split(new char[] { ' ', ':' });
                        port.localAddress = split2[7];
                        string deneme = JsonConvert.SerializeObject(port);
                        // Console.WriteLine(deneme);
                        Console.WriteLine("Port: {0}      Foreign Address: {1} ", port.localAddress, port.foreignAddress);
                        TextWriter yaziyaz = new StreamWriter(@"C:\Users\Süleyman\Desktop/jsonders.json", true);
                        yaziyaz.WriteLine(deneme);
                        yaziyaz.Close();
                    }


                }

                Console.ReadKey();
            }
        }
    }

    public class Json_veri
    {
        public string localAddress { get; set; }
        public string foreignAddress { get; set; }
    }
}
