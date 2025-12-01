using System;
using System.Collections.Generic;
using System.IO;
using System.Xml.Serialization;
using System.Text.Json;


[Serializable]
public class Group
{
public int Id;
public string Name;
public List<string> Members;
public bool IsActive;
public DateTime Created;
}


class Program
{
static void Main()
{
// создаём группу как студент :)
Group g = new Group();
g.Id = 1;
g.Name = "Група №1";
g.IsActive = true;
g.Created = DateTime.Now;
g.Members = new List<string>();
g.Members.Add("Петро");
g.Members.Add("Аня");
g.Members.Add("Олег");


// ---- БИНАРНАЯ СЕРИАЛИЗАЦИЯ ----
using (BinaryWriter bw = new BinaryWriter(File.Open("group.bin", FileMode.Create)))
{
bw.Write(g.Id);
bw.Write(g.Name);
bw.Write(g.IsActive);
bw.Write(g.Created.ToBinary());
bw.Write(g.Members.Count);
foreach (var m in g.Members) bw.Write(m);
}


// ---- XML ----
XmlSerializer xml = new XmlSerializer(typeof(Group));
using (FileStream fs = new FileStream("group.xml", FileMode.Create))
{
xml.Serialize(fs, g);
}


// ---- JSON ----
string json = JsonSerializer.Serialize(g);
File.WriteAllText("group.json", json);


// ======= ЧТЕНИЕ обратно =======
Group gBin;
using (BinaryReader br = new BinaryReader(File.Open("group.bin", FileMode.Open)))
{
gBin = new Group();
gBin.Id = br.ReadInt32();
gBin.Name = br.ReadString();
gBin.IsActive = br.ReadBoolean();
gBin.Created = DateTime.FromBinary(br.ReadInt64());
int c = br.ReadInt32();
gBin.Members = new List<string>();
for (int i = 0; i < c; i++) gBin.Members.Add(br.ReadString());
}


Group gXml;
using (FileStream fs = new FileStream("group.xml", FileMode.Open))
{
gXml = (Group)xml.Deserialize(fs);
}


Group gJson = JsonSerializer.Deserialize<Group>(File.ReadAllText("group.json"));


// выводим
Console.WriteLine("----- BIN -----");
Show(gBin);
Console.WriteLine("----- XML -----");
Show(gXml);
Console.WriteLine("----- JSON -----");
Show(gJson);
}


static void Show(Group g)
{
Console.WriteLine("Id:" + g.Id);
Console.WriteLine("Name:" + g.Name);
Console.WriteLine("Активна:" + g.IsActive);
Console.WriteLine("Дата:" + g.Created);
Console.WriteLine("Учасники:");
foreach (var m in g.Members) Console.WriteLine(" - " + m);
Console.WriteLine();
}
}
