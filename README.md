# graveyard-game"use client"

import { useState, useEffect } from "react"
import { Button } from "@/components/ui/button"
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription } from "@/components/ui/dialog"
import { toast } from "@/components/ui/use-toast"
import { Skull, Ghost, Wand2, Key, Candy, Flame, DoorOpen } from "lucide-react"
import { Creepster } from 'next/font/google'
import { motion, AnimatePresence } from "framer-motion"

const creepsterFont = Creepster({ weight: '400', subsets: ['latin'] })

export default function Component() {
  const [inventory, setInventory] = useState<string[]>([])
  const [isEscaped, setIsEscaped] = useState(false)
  const [showDialog, setShowDialog] = useState(false)
  const [dialogContent, setDialogContent] = useState({ title: "", description: "" })
  const [showJumpScare, setShowJumpScare] = useState(false)
  const [hasWinningItem, setHasWinningItem] = useState(false)

  const items = [
    { name: "Skull", icon: Skull },
    { name: "Ghost", icon: Ghost },
    { name: "WitchWand", icon: Wand2 },
    { name: "Key", icon: Key },
    { name: "Pumpkin", icon: Flame },
    { name: "Candy", icon: Candy },
  ]

  const curses = [
    "Oops! You've been turned into a pumpkin. Squash that idea and try again!",
    "Yikes! Your bones are now jelly. Wobble back to the start!",
    "Zap! You're now speaking in reverse. Escape to again try!",
    "Boo! You're invisible now. Reappear at the beginning!",
    "Croak! You're a frog prince/princess now. Hop back to square one!",
    "Poof! Your hair is now snakes. Slither back and try again!"
  ]

  useEffect(() => {
    const jumpScareTimer = setTimeout(() => {
      setShowJumpScare(true)
      setTimeout(() => setShowJumpScare(false), 500)
    }, 10000 + Math.random() * 20000) // Random time between 10-30 seconds

    return () => clearTimeout(jumpScareTimer)
  }, [])

  const addToInventory = (item: string) => {
    if (inventory.length < 6 && !inventory.includes(item)) {
      const newInventory = [...inventory, item]
      setInventory(newInventory)
      toast({
        title: `You picked up the ${item}!`,
        description: "It's been added to your inventory.",
      })
    
      // Check for potential win condition
      if (newInventory.length === 1 && ["Key", "Ghost", "WitchWand"].includes(item)) {
        setHasWinningItem(true)
      }

      // Check if all slots are filled
      if (newInventory.length === 6 && hasWinningItem) {
        setIsEscaped(true)
        setDialogContent({
          title: "You've Escaped!",
          description: `Congratulations! Your first item was the key to your escape. You've unlocked the cemetery gates. Enjoy your freedom!`,
        })
        setShowDialog(true)
      }
    } else if (inventory.includes(item)) {
      toast({
        title: `You already have the ${item}!`,
        description: "Try picking up something else.",
        variant: "destructive",
      })
    } else {
      toast({
        title: "Inventory is full!",
        description: "You can't carry any more items.",
        variant: "destructive",
      })
    }
  }

  const tryEscape = () => {
    if (inventory.length === 6 && hasWinningItem) {
      setIsEscaped(true)
      setDialogContent({
        title: "You've Escaped!",
        description: `Congratulations! Your first item was the key to your escape. You've unlocked the cemetery gates. Enjoy your freedom!`,
      })
    } else {
      const randomCurse = curses[Math.floor(Math.random() * curses.length)]
      setDialogContent({
        title: "Escape Failed",
        description: `${randomCurse} Play again to break the curse!`,
      })
    }
    setShowDialog(true)
  }

  const resetGame = () => {
    setInventory([])
    setIsEscaped(false)
    setShowDialog(false)
    setHasWinningItem(false)
  }

  return (
    <div className={`min-h-screen bg-cover bg-center flex flex-col items-center justify-center ${creepsterFont.className} relative overflow-hidden`}
         style={{backgroundImage: "url('https://hebbkx1anhila5yf.public.blob.vercel-storage.com/Screenshot%202024-10-23%20at%204.56.16%E2%80%AFPM-9YW13iDQ0dqV3HQhv52lJMLsADUUpQ.png')"}}>
      <div className="absolute inset-0 bg-black opacity-50"></div>
      
      <h1 
        className="glitch text-3xl sm:text-4xl md:text-5xl font-bold mb-4 text-purple-700 tracking-wider z-10 text-center w-full"
        data-text="Escape the Haunted Graveyard"
      >
        Escape the Haunted Graveyard
      </h1>
      <p className="mb-8 text-lg text-center text-purple-300 z-10 max-w-2xl">
        You find yourself trapped in a spooky graveyard.<br />
        Collect items and find the right combination to escape!
      </p>
      <div className="max-w-2xl w-full z-10 p-6">
        <div className="grid grid-cols-3 gap-2 mb-6">
          {items.map((item) => (
            <button
              key={item.name}
              onClick={() => addToInventory(item.name)}
              className="p-2 text-pink-500 hover:text-pink-400 transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-pink-400 focus:ring-opacity-50 rounded-lg"
              disabled={isEscaped || inventory.length >= 6}
            >
              <item.icon className="w-12 h-12 mx-auto animate-float" strokeWidth={1.5} />
            </button>
          ))}
        </div>
        <div className="mb-6">
          <div className="flex gap-1 justify-center">
            {[...Array(6)].map((_, index) => (
              <div key={index} className="w-8 h-8 rounded-full border-2 border-purple-900 flex items-center justify-center overflow-hidden bg-purple-950 bg-opacity-50">
                <AnimatePresence>
                  {inventory[index] && (
                    <motion.div
                      key={`${inventory[index]}-${index}`}
                      initial={{ y: -50, opacity: 0 }}
                      animate={{ y: 0, opacity: 1 }}
                      exit={{ y: 50, opacity: 0 }}
                      transition={{ duration: 0.5 }}
                      className="w-6 h-6 text-pink-500"
                    >
                      {(() => {
                        const item = items.find(item => item.name === inventory[index]);
                        return item ? <item.icon className="w-full h-full" strokeWidth={1.5} /> : null;
                      })()}
                    </motion.div>
                  )}
                </AnimatePresence>
              </div>
            ))}
          </div>
        </div>
        <Button
          onClick={tryEscape}
          className="px-4 py-2 text-lg bg-fuchsia-700 hover:bg-fuchsia-600 tracking-wider text-purple-300 relative overflow-hidden group mx-auto flex items-center"
          disabled={isEscaped || inventory.length < 6}
        >
          <DoorOpen className="w-5 h-5 mr-2 animate-bounce text-purple-300" strokeWidth={1.5} />
          Try to Escape
          <div className="absolute inset-0 bg-purple-500 opacity-0 group-hover:opacity-20 transition-opacity duration-300" />
        </Button>
      </div>
      <Dialog open={showDialog} onOpenChange={setShowDialog}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>{dialogContent.title}</DialogTitle>
            <DialogDescription>{dialogContent.description}</DialogDescription>
          </DialogHeader>
          <Button onClick={resetGame} className="mt-4">
            {isEscaped ? "Play Again" : "Try Again"}
          </Button>
        </DialogContent>
      </Dialog>

      {/* Jump Scare */}
      {showJumpScare && (
        <div className="fixed inset-0 bg-black z-50 flex items-center justify-center">
          <img src="/placeholder.svg?height=300&width=300" alt="Scary face" className="w-1/2 h-1/2 object-contain animate-pulse" />
        </div>
      )}
    </div>
  )
}

<style jsx>{`
@keyframes glitch {
  0% {
    transform: translate(0);
  }
  20% {
    transform: translate(-5px, 5px);
  }
  40% {
    transform: translate(-5px, -5px);
  }
  60% {
    transform: translate(5px, 5px);
  }
  80% {
    transform: translate(5px, -5px);
  }
  100% {
    transform: translate(0);
  }
}

.glitch {
  position: relative;
}

.glitch::before,
.glitch::after {
  content: attr(data-text);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.glitch::before {
  left: 2px;
  text-shadow: -2px 0 #ff00c1;
  clip: rect(44px, 450px, 56px, 0);
  animation: glitch 5s infinite linear alternate-reverse;
}

.glitch::after {
  left: -2px;
  text-shadow: -2px 0 #00fff9, 2px 2px #ff00c1;
  animation: glitch 1s infinite linear alternate-reverse;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.animate-float {
  animation: float 3s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.animate-pulse {
  animation: pulse 2s ease-in-out infinite;
}
`}</style>
