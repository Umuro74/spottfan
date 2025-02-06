import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Avatar } from "@/components/ui/avatar";
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";
import { MessageSquare, Trophy, ShoppingCart, Users, Globe, Video, Mic, Calendar, MapPin } from "lucide-react";
import { auth, db } from "../firebaseConfig";
import { signInWithPopup, GoogleAuthProvider } from "firebase/auth";
import { collection, addDoc, onSnapshot } from "firebase/firestore";

export default function SportsFanPlatform() {
  const [user, setUser] = useState(null);
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState("");
  const [meetups, setMeetups] = useState([]);
  const [newMeetup, setNewMeetup] = useState("");

  useEffect(() => {
    const unsubscribeMessages = onSnapshot(collection(db, "messages"), (snapshot) => {
      setMessages(snapshot.docs.map(doc => doc.data().text));
    });

    const unsubscribeMeetups = onSnapshot(collection(db, "meetups"), (snapshot) => {
      setMeetups(snapshot.docs.map(doc => doc.data()));
    });

    return () => {
      unsubscribeMessages();
      unsubscribeMeetups();
    };
  }, []);

  const handleSignIn = async () => {
    const provider = new GoogleAuthProvider();
    const result = await signInWithPopup(auth, provider);
    setUser(result.user);
  };

  const handleSendMessage = async () => {
    if (newMessage.trim()) {
      await addDoc(collection(db, "messages"), { text: newMessage });
      setNewMessage("");
    }
  };

  const handleAddMeetup = async () => {
    if (newMeetup.trim()) {
      await addDoc(collection(db, "meetups"), { match: "Custom Meetup", location: newMeetup });
      setNewMeetup("");
    }
  };

  return (
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold text-center mb-6">Sports Fan Rivalry Platform</h1>
      {user ? (
        <Card className="mb-6 p-4 flex items-center space-x-4">
          <Avatar className="w-16 h-16" src={user.photoURL} />
          <div>
            <h2 className="text-xl font-bold">{user.displayName}</h2>
            <p className="text-gray-500">{user.email}</p>
          </div>
        </Card>
      ) : (
        <Button className="mb-4 w-full" onClick={handleSignIn}>Sign in with Google</Button>
      )}
      
      <Tabs defaultValue="chat">
        <TabsList className="flex justify-around bg-gray-100 p-2 rounded-lg">
          <TabsTrigger value="chat"><MessageSquare className="w-5 h-5" /> Chat</TabsTrigger>
          <TabsTrigger value="meetups"><MapPin className="w-5 h-5" /> Meetups</TabsTrigger>
        </TabsList>

        <TabsContent value="chat">
          <Card className="p-4 mt-4">
            <h3 className="text-lg font-bold">Live Match Chat</h3>
            <div className="mt-4 space-y-2">
              {messages.map((msg, index) => (
                <p key={index}><strong>Fan:</strong> {msg}</p>
              ))}
            </div>
            {user && (
              <>
                <Input className="mt-4" placeholder="Type your message..." value={newMessage} onChange={(e) => setNewMessage(e.target.value)} />
                <Button className="mt-2 w-full" onClick={handleSendMessage}>Send</Button>
              </>
            )}
          </Card>
        </TabsContent>

        <TabsContent value="meetups">
          <Card className="p-4 mt-4">
            <h3 className="text-lg font-bold">Fan Meetups</h3>
            <p>Organize and join meetups before attending matches.</p>
            <div className="mt-4 space-y-2">
              {meetups.map((meetup, index) => (
                <p key={index}><strong>{meetup.match}:</strong> {meetup.location}</p>
              ))}
            </div>
            {user && (
              <>
                <Input className="mt-4" placeholder="Suggest a new meetup location..." value={newMeetup} onChange={(e) => setNewMeetup(e.target.value)} />
                <Button className="mt-2 w-full" onClick={handleAddMeetup}>Submit</Button>
              </>
            )}
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}
