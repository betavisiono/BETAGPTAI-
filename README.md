"use client";
import React from "react";

import {
  useUpload,
  useHandleStreamResponse,
} from "../utilities/runtime-helpers";

function MainComponent() {
  const [selectedAI, setSelectedAI] = useState("chat");
  const [userInput, setUserInput] = useState("");
  const [messages, setMessages] = useState([]);
  const [loading, setLoading] = useState(false);
  const [streamingMessage, setStreamingMessage] = useState("");
  const [error, setError] = useState("");
  const [upload, { loading: uploadLoading }] = useUpload();
  const [uploadedImage, setUploadedImage] = useState(null);
  const [menuOpen, setMenuOpen] = useState(false);
  const [profileMenuOpen, setProfileMenuOpen] = useState(false);
  const { data: user, loading: userLoading } = useUser();
  const handleStreamResponse = useHandleStreamResponse({
    onChunk: setStreamingMessage,
    onFinish: (message) => {
      setMessages((prev) => [...prev, { role: "assistant", content: message }]);
      setStreamingMessage("");
      setLoading(false);
    },
  });
  const handleFileUpload = async (e) => {
    const file = e.target.files[0];
    if (file) {
      const { url, error } = await upload({ file });
      if (error) {
        setError(error);
        return;
      }
      setUploadedImage(url);
    }
  };
  const handleSubmit = async () => {
    if (!userInput.trim() && !uploadedImage) return;

    setLoading(true);
    setError("");

    const newMessage = { role: "user", content: userInput };
    setMessages((prev) => [...prev, newMessage]);
    setUserInput("");

    try {
      let endpoint = "/integrations/chat-gpt/conversationgpt4";
      let body = { messages: [...messages, newMessage], stream: true };
      let method = "POST";

      switch (selectedAI) {
        case "vision":
          if (uploadedImage) {
            endpoint = "/integrations/gpt-vision/";
            body = {
              messages: [
                {
                  role: "user",
                  content: [
                    { type: "text", text: userInput },
                    { type: "image_url", image_url: { url: uploadedImage } },
                  ],
                },
              ],
            };
          }
          break;
        case "dalle":
          endpoint = `/integrations/dall-e-3/?prompt=${encodeURIComponent(
            userInput
          )}`;
          method = "GET";
          break;
        case "moderation":
          endpoint = "/integrations/text-moderation/";
          body = { input: userInput };
          break;
        case "translate":
          endpoint = "/integrations/google-translate/language/translate/v2";
          method = "POST";
          body = new URLSearchParams({
            q: userInput,
            target: "fr",
          });
          break;
      }

      const response = await fetch(endpoint, {
        method,
        headers:
          method === "POST" &&
          endpoint !== "/integrations/google-translate/language/translate/v2"
            ? { "Content-Type": "application/json" }
            : undefined,
        body:
          method === "POST"
            ? endpoint ===
              "/integrations/google-translate/language/translate/v2"
              ? body
              : JSON.stringify(body)
            : undefined,
      });

      handleStreamResponse(response);
    } catch (err) {
      setError("Une erreur est survenue");
      setLoading(false);
    }
  };

  return (
    <div className="flex flex-col h-screen bg-[#000033]">
      <header className="flex items-center justify-between p-6 border-b border-[#00BFFF]/20">
        <div className="flex items-center">
          <div className="relative">
            <button
              onClick={() => setMenuOpen(!menuOpen)}
              className="text-[#00BFFF] hover:text-[#33CCFF] p-2"
            >
              <i className="fas fa-robot text-xl" />
            </button>
            {menuOpen && (
              <div className="absolute top-full left-0 mt-2 w-64 bg-[#000044] rounded-lg border border-[#00BFFF]/20 shadow-lg z-50">
                <div className="p-2 space-y-1">
                  <button
                    onClick={() => setSelectedAI("chat")}
                    className={`w-full flex items-center px-4 py-3 rounded-lg ${
                      selectedAI === "chat"
                        ? "bg-[#00BFFF] text-[#000033]"
                        : "text-[#00BFFF] hover:bg-[#000066]"
                    }`}
                  >
                    <i className="fas fa-comments mr-3" />
                    ChatGPT
                  </button>
                  <button
                    onClick={() => setSelectedAI("dalle")}
                    className={`w-full flex items-center px-4 py-3 rounded-lg ${
                      selectedAI === "dalle"
                        ? "bg-[#00BFFF] text-[#000033]"
                        : "text-[#00BFFF] hover:bg-[#000066]"
                    }`}
                  >
                    <i className="fas fa-paint-brush mr-3" />
                    DALL-E 3
                  </button>
                  <button
                    onClick={() => setSelectedAI("vision")}
                    className={`w-full flex items-center px-4 py-3 rounded-lg ${
                      selectedAI === "vision"
                        ? "bg-[#00BFFF] text-[#000033]"
                        : "text-[#00BFFF] hover:bg-[#000066]"
                    }`}
                  >
                    <i className="fas fa-eye mr-3" />
                    GPT Vision
                  </button>
                  <button
                    onClick={() => setSelectedAI("moderation")}
                    className={`w-full flex items-center px-4 py-3 rounded-lg ${
                      selectedAI === "moderation"
                        ? "bg-[#00BFFF] text-[#000033]"
                        : "text-[#00BFFF] hover:bg-[#000066]"
                    }`}
                  >
                    <i className="fas fa-shield-alt mr-3" />
                    Modération de texte
                  </button>
                </div>
              </div>
            )}
          </div>
        </div>
        <div className="flex items-center space-x-8">
          <img
            src="https://ucarecdn.com/f79e7918-944a-4e22-bd2f-7cb69681253a/-/format/auto/"
            alt="BetaGPTai Logo"
            className="w-12 h-12"
          />
          <h1 className="text-[#00BFFF] text-2xl font-bold">BETAGPTAI</h1>
          <div className="flex items-center space-x-4">
            <a
              href="https://BetaVisionO.home.blog/"
              target="_blank"
              rel="noopener noreferrer"
              className="px-4 py-2 rounded-lg bg-[#00BFFF] text-[#000033] hover:bg-[#33CCFF] transition-colors"
            >
              BetaVisionO
            </a>
            <a
              href="https://betasocials.bettermode.io/"
              target="_blank"
              rel="noopener noreferrer"
              className="px-4 py-2 rounded-lg bg-[#00BFFF] text-[#000033] hover:bg-[#33CCFF] transition-colors"
            >
              BetaSocial
            </a>
          </div>
        </div>
        <div className="relative">
          <button
            onClick={() => setProfileMenuOpen(!profileMenuOpen)}
            className="text-[#00BFFF] hover:text-[#33CCFF]"
          >
            <i className="fas fa-user-circle text-xl" />
          </button>
          {profileMenuOpen && (
            <div className="absolute top-full right-0 mt-2 w-64 bg-[#000044] rounded-lg border border-[#00BFFF]/20 shadow-lg z-50">
              {!user ? (
                <>
                  <a
                    href="/account/signin"
                    className="flex items-center px-4 py-3 text-[#00BFFF] hover:bg-[#000066] rounded-t-lg"
                  >
                    <i className="fas fa-sign-in-alt mr-2" />
                    Se connecter
                  </a>
                  <a
                    href="/account/signup"
                    className="flex items-center px-4 py-3 text-[#00BFFF] hover:bg-[#000066] rounded-b-lg"
                  >
                    <i className="fas fa-user-plus mr-2" />
                    S'inscrire
                  </a>
                </>
              ) : (
                <>
                  <div className="px-4 py-3 text-[#00BFFF] border-b border-[#00BFFF]/20">
                    <div className="font-bold">
                      {user.name || "Utilisateur"}
                    </div>
                    <div className="text-sm opacity-75">{user.email}</div>
                  </div>
                  <a
                    href="/account/profile"
                    className="flex items-center px-4 py-3 text-[#00BFFF] hover:bg-[#000066]"
                  >
                    <i className="fas fa-user mr-2" />
                    Voir mon profil
                  </a>
                  <a
                    href="/account/settings"
                    className="flex items-center px-4 py-3 text-[#00BFFF] hover:bg-[#000066]"
                  >
                    <i className="fas fa-cog mr-2" />
                    Paramètres du compte
                  </a>
                  <a
                    href="/account/logout"
                    className="flex items-center px-4 py-3 text-[#00BFFF] hover:bg-[#000066] rounded-b-lg"
                  >
                    <i className="fas fa-sign-out-alt mr-2" />
                    Se déconnecter
                  </a>
                </>
              )}
            </div>
          )}
        </div>
      </header>

      <div className="flex-1 overflow-auto p-6 space-y-4">
        {messages.map((message, index) => (
          <div
            key={index}
            className={`flex ${
              message.role === "user" ? "justify-end" : "justify-start"
            }`}
          >
            <div
              className={`max-w-[70%] p-4 rounded-lg ${
                message.role === "user"
                  ? "bg-[#00BFFF] text-[#000033]"
                  : "bg-[#000066] text-white"
              }`}
            >
              {message.content}
            </div>
          </div>
        ))}
        {streamingMessage && (
          <div className="flex justify-start">
            <div className="max-w-[70%] p-4 rounded-lg bg-[#000066] text-white">
              {streamingMessage}
            </div>
          </div>
        )}
        {loading && (
          <div className="flex justify-center">
            <div className="w-8 h-8 border-4 border-[#00BFFF] border-t-transparent rounded-full animate-spin" />
          </div>
        )}
      </div>

      <div className="p-6 bg-[#000044] border-t border-[#00BFFF]/20">
        {error && <div className="text-red-500 mb-4">{error}</div>}
        <div className="flex items-end space-x-4">
          {(selectedAI === "vision" || selectedAI === "dalle") && (
            <label className="flex items-center justify-center w-12 h-12 rounded-lg bg-[#000066] hover:bg-[#000088] cursor-pointer">
              <input
                type="file"
                accept="image/*"
                onChange={handleFileUpload}
                className="hidden"
              />
              <i className="fas fa-upload text-[#00BFFF]" />
            </label>
          )}
          <textarea
            value={userInput}
            onChange={(e) => setUserInput(e.target.value)}
            placeholder="Écrivez votre message..."
            className="flex-1 p-4 rounded-lg bg-[#000066] text-white border-2 border-[#00BFFF] focus:outline-none focus:border-[#FF0000] resize-none"
            rows="1"
          />
          <button
            onClick={handleSubmit}
            disabled={loading}
            className="w-12 h-12 rounded-lg bg-[#00BFFF] text-[#000033] hover:bg-[#33CCFF] flex items-center justify-center"
          >
            <i className="fas fa-paper-plane" />
          </button>
        </div>
      </div>
      <style jsx global>{`
        @keyframes slideIn {
          from {
            opacity: 0;
            transform: translateY(10px);
          }
          to {
            opacity: 1;
            transform: translateY(0);
          }
        }

        .flex-1 > div {
          animation: slideIn 0.3s ease-out;
        }
      `}</style>
    </div>
  );
}

export default MainComponent;
