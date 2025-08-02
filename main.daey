import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:video_player/video_player.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_storage/firebase_storage.dart';
import 'dart:io';
import 'package:image_picker/image_picker.dart';

// ------------------ MAIN ------------------
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Watch & Earn',
      theme: ThemeData.dark(),
      debugShowCheckedModeBanner: false,
      home: FirebaseAuth.instance.currentUser == null
          ? LoginPage()
          : HomePage(),
    );
  }
}

// ------------------ LOGIN / REGISTER ------------------
class LoginPage extends StatefulWidget {
  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  TextEditingController email = TextEditingController();
  TextEditingController pass = TextEditingController();
  bool isLogin = true;

  Future<void> authAction() async {
    try {
      if (isLogin) {
        await FirebaseAuth.instance.signInWithEmailAndPassword(
          email: email.text.trim(),
          password: pass.text.trim(),
        );
      } else {
        await FirebaseAuth.instance.createUserWithEmailAndPassword(
          email: email.text.trim(),
          password: pass.text.trim(),
        );
        // Save user info
        await FirebaseFirestore.instance
            .collection('users')
            .doc(FirebaseAuth.instance.currentUser!.uid)
            .set({
          'coins': 0,
          'membership': false,
        });
      }
      setState(() {});
    } catch (e) {
      ScaffoldMessenger.of(context)
          .showSnackBar(SnackBar(content: Text(e.toString())));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: EdgeInsets.all(20),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(isLogin ? "Login" : "Register", style: TextStyle(fontSize: 28)),
            TextField(controller: email, decoration: InputDecoration(labelText: "Email")),
            TextField(controller: pass, decoration: InputDecoration(labelText: "Password"), obscureText: true),
            SizedBox(height: 20),
            ElevatedButton(onPressed: authAction, child: Text(isLogin ? "Login" : "Register")),
            TextButton(
              onPressed: () => setState(() => isLogin = !isLogin),
              child: Text(isLogin ? "Create Account" : "Have account? Login"),
            )
          ],
        ),
      ),
    );
  }
}

// ------------------ HOME PAGE ------------------
class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int coins = 0;
  bool membership = false;

  @override
  void initState() {
    super.initState();
    loadUser();
  }

  Future<void> loadUser() async {
    var snap = await FirebaseFirestore.instance
        .collection('users')
        .doc(FirebaseAuth.instance.currentUser!.uid)
        .get();
    setState(() {
      coins = snap['coins'];
      membership = snap['membership'];
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Watch & Earn"),
        actions: [
          IconButton(
            icon: Icon(Icons.logout),
            onPressed: () => FirebaseAuth.instance.signOut().then(
                  (_) => Navigator.pushReplacement(
                    context,
                    MaterialPageRoute(builder: (_) => LoginPage()),
                  ),
                ),
          )
        ],
      ),
      body: membership
          ? Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text("💰 Coins: $coins", style: TextStyle(fontSize: 24)),
                  ElevatedButton(
                    onPressed: () => Navigator.push(
                      context,
                      MaterialPageRoute(builder: (_) => VideoPage()),
                    ).then((_) => loadUser()),
                    child: Text("🎥 Watch Video"),
                  ),
                  ElevatedButton(
                    onPressed: () => Navigator.push(
                      context,
                      MaterialPageRoute(builder: (_) => WithdrawPage()),
                    ),
                    child: Text("💵 Withdraw"),
                  )
                ],
              ),
            )
          : PaymentPage(),
    );
  }
}

// ------------------ VIDEO PAGE ------------------
class VideoPage extends StatefulWidget {
  @override
  _VideoPageState createState() => _VideoPageState();
}

class _VideoPageState extends State<VideoPage> {
  late VideoPlayerController _controller;
  int lastSecond = 0;
  int earnedCoins = 0;

  @override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.network(
      "https://samplelib.com/lib/preview/mp4/sample-5s.mp4", // Replace with MP4 link
    )..initialize().then((_) {
        setState(() {});
        _controller.play();
      });

    _controller.addListener(() {
      final sec = _controller.value.position.inSeconds;
      if (sec > lastSecond) {
        lastSecond = sec;
        earnedCoins++;
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> saveCoins() async {
    var ref = FirebaseFirestore.instance
        .collection('users')
        .doc(FirebaseAuth.instance.currentUser!.uid);
    var snap = await ref.get();
    int current = snap['coins'];
    await ref.update({'coins': current + earnedCoins});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Watch Video")),
      body: Center(
        child: _controller.value.isInitialized
            ? AspectRatio(
                aspectRatio: _controller.value.aspectRatio,
                child: VideoPlayer(_controller),
              )
            : CircularProgressIndicator(),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () async {
          await saveCoins();
          Navigator.pop(context);
        },
        child: Icon(Icons.check),
      ),
    );
  }
}

// ------------------ PAYMENT PAGE ------------------
class PaymentPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text("Pay 2000Ks to Join", style: TextStyle(fontSize: 22)),
          SizedBox(height: 20),
          Image.asset("assets/wave_qr.png", height: 150), // Wave QR
          Image.asset("assets/kbz_qr.png", height: 150),  // KBZ QR
          SizedBox(height: 20),
          Text("Upload payment screenshot"),
          ElevatedButton(
            onPressed: () async {
              final picker = ImagePicker();
              final picked = await picker.pickImage(source: ImageSource.gallery);
              if (picked != null) {
                File file = File(picked.path);
                await FirebaseStorage.instance
                    .ref("payments/${FirebaseAuth.instance.currentUser!.uid}.jpg")
                    .putFile(file);
                ScaffoldMessenger.of(context)
                    .showSnackBar(SnackBar(content: Text("Uploaded, wait for approval")));
              }
            },
            child: Text("Upload Screenshot"),
          )
        ],
      ),
    );
  }
}

// ------------------ WITHDRAW PAGE ------------------
class WithdrawPage extends StatefulWidget {
  @override
  _WithdrawPageState createState() => _WithdrawPageState();
}

class _WithdrawPageState extends State<WithdrawPage> {
  TextEditingController number = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Withdraw")),
      body: Padding(
        padding: EdgeInsets.all(20),
        child: Column(
          children: [
            TextField(controller: number, decoration: InputDecoration(labelText: "Wave/KBZ Number")),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                await FirebaseFirestore.instance.collection('withdraws').add({
                  'uid': FirebaseAuth.instance.currentUser!.uid,
                  'number': number.text,
                  'status': 'pending'
                });
                ScaffoldMessenger.of(context)
                    .showSnackBar(SnackBar(content: Text("Withdraw request sent")));
              },
              child: Text("Send Request"),
            )
          ],
        ),
      ),
    );
  }
}
