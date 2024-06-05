Privacy Policy  
----------------

### Introduction  
Our privacy policy will help you understand what information we collect at MtiSoft app, how MtiSoft app uses it, and what choices you have.
MtiSoft app built the MtiSoft app as a free app. This SERVICE is provided by MtiSoft app at no cost and is intended for use as is.
If you choose to use our Service, then you agree to the collection and use of information in  relation with this policy. The Personal Information that we collect are used for providing and improving the Service. We will not use or share your information with anyone except as described in this Privacy Policy.  
The terms used in this Privacy Policy have the same meanings as in our Terms and Conditions, which is accessible in our website, unless otherwise  defined in this Privacy Policy.

### Information Collection and Use  
For a better experience while using our Service, we may require you to provide us with certain personally identifiable information, including but not limited to users name, email address, gender, location, pictures. The information that we request will be retained by us and used as described in this privacy policy.  
The app does use third party services that may collect information used to identify you. 

### Cookies  
Cookies are files with small amount of data that is commonly used an anonymous unique identifier. These are sent to your browser from the website that you visit and are stored on your devices’s internal memory.  

This Services does not uses these “cookies” explicitly. However, the app may use third party code and libraries that use “cookies” to collection information and to improve their services. You have the option  to either accept or refuse these cookies, and know when a cookie is being sent to your device. If you choose to refuse our cookies, you may not be able to use some portions of this Service.  

### Location Information  
Some of the services may use location information transmitted from users' mobile phones. We only use this information within the scope necessary for the designated service.  

### Device Information  
We collect information from your device in some cases. The information will be utilized for the provision of better service and to prevent fraudulent acts. Additionally, such information will not include that which will identify the individual user.  

### Service Providers  
We may employ third-party companies and individuals due to the following reasons:  
* To facilitate our Service;
* To provide the Service on our behalf;
* To perform Service-related services; or
* To assist us in analyzing how our Service is used.  

We want to inform users of this Service that these third parties have access to your Personal Information. The reason is to perform the tasks assigned to them on our behalf. However, they are obligated not to disclose or use the information for any other purpose.  

### Security  
We value your trust in providing us your Personal Information, thus we are striving to use commercially acceptable means of protecting it. But remember that no method of transmission over  the internet, or method of electronic storage is 100% secure and reliable, and we cannot guarantee its absolute security.  

### Children’s Privacy  
This Services do not address anyone under the age of 13. We do not knowingly collect personal identifiable information from children under 13. In the case we discover that a child under 13 has provided us with personal information, we immediately delete this from our servers. If you  are  a  parent  or  guardian and you are aware that your child has provided us with personal information, please contact us so that we will be able to do necessary actions.  

### Changes to This Privacy Policy  
We may update our Privacy Policy from time to time. Thus, you are advised to review this page periodically for any changes. We will notify you of any changes by posting the new Privacy Policy on this page. These changes are effective immediately, after they are posted on this page.  

### Contact Us  
If you have any questions or suggestions about our Privacy Policy, do not hesitate to contact us.  
Contact Information:  
Email:support@mtideg.edu.in




import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static final DatabaseHelper _instance = DatabaseHelper._internal();
  factory DatabaseHelper() => _instance;

  static Database? _database;

  DatabaseHelper._internal();

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  Future<Database> _initDatabase() async {
    String path = join(await getDatabasesPath(), 'rewards.db');
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE rewards(
        id INTEGER PRIMARY KEY,
        date TEXT,
        points INTEGER,
        type TEXT,
        agentid TEXT
      )
    ''');
  }

  // Insert reward
  Future<void> insertReward(Map<String, dynamic> reward) async {
    final db = await database;
    await db.insert('rewards', reward);
  }

  // Get rewards by date range
  Future<List<Map<String, dynamic>>> getRewardsByDateRange(DateTime fromDate, DateTime toDate) async {
    final db = await database;
    String from = fromDate.toIso8601String().split('T').first;
    String to = toDate.toIso8601String().split('T').first;
    return await db.query(
      'rewards',
      where: 'date BETWEEN ? AND ?',
      whereArgs: [from, to],
      orderBy: 'date ASC'
    );
  }

  // Update reward
  Future<void> updateReward(Map<String, dynamic> reward) async {
    final db = await database;
    await db.update('rewards', reward, where: 'id = ?', whereArgs: [reward['id']]);
  }

  // Delete reward
  Future<void> deleteReward(int id) async {
    final db = await database;
    await db.delete('rewards', where: 'id = ?', whereArgs: [id]);
  }
}



import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'database_helper.dart';
import 'reward.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Rewards Report',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: RewardsReportPage(),
    );
  }
}

class RewardsReportPage extends StatefulWidget {
  @override
  _RewardsReportPageState createState() => _RewardsReportPageState();
}

class _RewardsReportPageState extends State<RewardsReportPage> {
  final DatabaseHelper _dbHelper = DatabaseHelper();
  DateTime? _fromDate;
  DateTime? _toDate;
  List<Reward> _rewards = [];

  void _pickFromDate() async {
    DateTime? picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(2000),
      lastDate: DateTime(2101),
    );
    if (picked != null) {
      setState(() {
        _fromDate = picked;
      });
    }
  }

  void _pickToDate() async {
    DateTime? picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(2000),
      lastDate: DateTime(2101),
    );
    if (picked != null) {
      setState(() {
        _toDate = picked;
      });
    }
  }

  void _generateReport() async {
    if (_fromDate != null && _toDate != null) {
      List<Map<String, dynamic>> rawRewards = await _dbHelper.getRewardsByDateRange(_fromDate!, _toDate!);
      setState(() {
        _rewards = rawRewards.map((rewardMap) => Reward.fromMap(rewardMap)).toList();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Rewards Report'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                TextButton(
                  onPressed: _pickFromDate,
                  child: Text(_fromDate == null ? 'Select From Date' : DateFormat.yMd().format(_fromDate!)),
                ),
                TextButton(
                  onPressed: _pickToDate,
                  child: Text(_toDate == null ? 'Select To Date' : DateFormat.yMd().format(_toDate!)),
                ),
                ElevatedButton(
                  onPressed: _generateReport,
                  child: Text('Generate Report'),
                ),
              ],
            ),
            SizedBox(height: 20),
            Expanded(
              child: _rewards.isEmpty
                  ? Center(child: Text('No data available'))
                  : DataTable(
                      columns: [
                        DataColumn(label: Text('S.No')),
                        DataColumn(label: Text('Date')),
                        DataColumn(label: Text('Reward Points')),
                      ],
                      rows: _rewards
                          .asMap()
                          .entries
                          .map(
                            (entry) => DataRow(cells: [
                              DataCell(Text((entry.key + 1).toString())),
                              DataCell(Text(entry.value.date)),
                              DataCell(Text(entry.value.points.toString())),
                            ]),
                          )
                          .toList(),
                    ),
            ),
          ],
        ),
      ),
    );
  }
}





Future<List<Reward>> getRewardsByDateRange(DateTime fromDate, DateTime toDate) async {
  final db = await database;
  String from = fromDate.toIso8601String().split('T').first;
  String to = toDate.toIso8601String().split('T').first;
  final List<Map<String, dynamic>> maps = await db.query(
    'rewards',
    where: 'date BETWEEN ? AND ?',
    whereArgs: [from, to],
    orderBy: 'date ASC'
  );

  // Convert List<Map<String, dynamic>> to List<Reward>
  return List.generate(maps.length, (i) {
    return Reward.fromMap(maps[i]);
  });
}






import 'dart:io';
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:path_provider/path_provider.dart';
import 'package:pdf/widgets.dart' as pw;
import 'package:open_filex/open_filex.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:excel/excel.dart';
import 'database_helper.dart';
import 'reward.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Rewards Report',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: RewardsReportPage(),
    );
  }
}

class RewardsReportPage extends StatefulWidget {
  @override
  _RewardsReportPageState createState() => _RewardsReportPageState();
}

class _RewardsReportPageState extends State<RewardsReportPage> {
  final DatabaseHelper _dbHelper = DatabaseHelper();
  DateTime? _fromDate;
  DateTime? _toDate;
  List<Reward> _rewards = [];

  void _pickFromDate() async {
    DateTime? picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(2000),
      lastDate: DateTime(2101),
    );
    if (picked != null) {
      setState(() {
        _fromDate = picked;
      });
    }
  }

  void _pickToDate() async {
    DateTime? picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(2000),
      lastDate: DateTime(2101),
    );
    if (picked != null) {
      setState(() {
        _toDate = picked;
      });
    }
  }

  void _generateReport() async {
    if (_fromDate != null && _toDate != null) {
      List<Reward> rewards = await _dbHelper.getRewardsByDateRange(_fromDate!, _toDate!);
      setState(() {
        _rewards = rewards;
      });
    }
  }

  Future<void> _generatePDF() async {
    if (await _requestStoragePermission()) {
      final pdf = pw.Document();

      pdf.addPage(
        pw.Page(
          build: (pw.Context context) => pw.Column(
            children: [
              pw.Text('Rewards Report', style: pw.TextStyle(fontSize: 24)),
              pw.SizedBox(height: 20),
              pw.Table.fromTextArray(
                headers: ['S.No', 'Date', 'Reward Points'],
                data: List<List<String>>.generate(
                  _rewards.length,
                  (index) => [
                    (index + 1).toString(),
                    _rewards[index].date,
                    _rewards[index].points.toString(),
                  ],
                ),
              ),
            ],
          ),
        ),
      );

      final directory = await getExternalStorageDirectory();
      if (directory != null) {
        final filePath = await _getUniqueFilePath(directory.path, 'rewards_report', 'pdf');
        final file = File(filePath);
        await file.writeAsBytes(await pdf.save());
        OpenFilex.open(file.path);
      }
    } else {
      // Handle permission denied
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Storage permission is required to save the file')),
      );
    }
  }

  Future<void> _generateExcel() async {
    if (await _requestStoragePermission()) {
      final excel = Excel.createExcel();
      final sheet = excel['Sheet1'];

      // Adding header row
      sheet.appendRow(['S.No', 'Date', 'Reward Points']);

      // Adding data rows
      for (int i = 0; i < _rewards.length; i++) {
        sheet.appendRow([
          (i + 1).toString(),
          _rewards[i].date,
          _rewards[i].points.toString(),
        ]);
      }

      final directory = await getExternalStorageDirectory();
      if (directory != null) {
        final filePath = await _getUniqueFilePath(directory.path, 'rewards_report', 'xlsx');
        final file = File(filePath);
        final bytes = excel.encode()!;
        await file.writeAsBytes(bytes);
        OpenFilex.open(file.path);
      }
    } else {
      // Handle permission denied
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Storage permission is required to save the file')),
      );
    }
  }

  Future<bool> _requestStoragePermission() async {
    PermissionStatus status = await Permission.storage.request();
    return status.isGranted;
  }

  Future<String> _getUniqueFilePath(String basePath, String fileName, String extension) async {
    int counter = 1;
    String fullPath = '$basePath/$fileName.$extension';

    while (await File(fullPath).exists()) {
      fullPath = '$basePath/$fileName($counter).$extension';
      counter++;
    }

    return fullPath;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Rewards Report'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                TextButton(
                  onPressed: _pickFromDate,
                  child: Text(_fromDate == null ? 'Select From Date' : DateFormat.yMd().format(_fromDate!)),
                ),
                TextButton(
                  onPressed: _pickToDate,
                  child: Text(_toDate == null ? 'Select To Date' : DateFormat.yMd().format(_toDate!)),
                ),
                ElevatedButton(
                  onPressed: _generateReport,
                  child: Text('Generate Report'),
                ),
              ],
            ),
            SizedBox(height: 20),
            Expanded(
              child: _rewards.isEmpty
                  ? Center(child: Text('No data available'))
                  : DataTable(
                      columns: [
                        DataColumn(label: Text('S.No')),
                        DataColumn(label: Text('Date')),
                        DataColumn(label: Text('Reward Points')),
                      ],
                      rows: _rewards
                          .asMap()
                          .entries
                          .map(
                            (entry) => DataRow(cells: [
                              DataCell(Text((entry.key + 1).toString())),
                              DataCell(Text(entry.value.date)),
                              DataCell(Text(entry.value.points.toString())),
                            ]),
                          )
                          .toList(),
                    ),
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: _generatePDF,
                  child: Text('Generate PDF'),
                ),
                ElevatedButton(
                  onPressed: _generateExcel,
                  child: Text('Generate Excel'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}




Future<String> _getUniqueFilePath(String basePath, String fileName, String extension) async {
  int counter = 1;
  String fullPath = '$basePath/$fileName.$extension';

  while (await File(fullPath).exists()) {
    fullPath = '$basePath/$fileName($counter).$extension';
    counter++;
  }

  return fullPath;
}
