## Quests

This package for KitRpg includes the basics for all quest behaviour contained in KitRpg. It has been made to be as customizable as possible. This allow you (and other packs) to create new Quest Objectives and starting conditions.

### Defines

The defines included are the "Quests" define, which simply state wether the quests should be enabled or not, and the "Simple Quest NPCs" define, which enable or disable two simple components that allow you to give and finish a quest on interaction (it's very basic, just good for testing or as an example to implement some quest giving behaviour).

### Compatibility

For now, it is compatible with (== use features of) the Data Saving pack.

### Usage

You can create new quests by using the create button in the project tab. Each quest object created like this **MUST BE INSIDE A "Resources" FOLDER**. This is very important, since otherwise, it won't be added into the game.

Each quest contains two main arrays : The starting conditions, and the objectives. A quest can only start if all starting conditions are met, and can only be finished if all objectives are done.

### UI

This packs adds two UI elements, one window, and one item. The QuestWindow is a "classical" window, that you can edit ot your liking. The item is the QuestItem, which is added, one per quest, into the "container" field of the window.

Both objects can be found as prefabs inside the "KitRpg/Quests/Prefabs" folder.

### Developpers

Here are some tips to expand the quests package : 

#### New quest objective

The quest objectives are somewhat restrictive, since their editor has been changed to allow multiple types. Therefore, the only fields the player can fill are of type string (for now at least). Let's start with the basic. To add a condition type, you need to modify the partial class QuestObjective : 

    public partial class QuestObjective
    {
        static void AddObjectiveTypes_MyExtensionName()
        {
            // This will add it into the dropdown menu inside of the quest template inspector
            existingTypes.Add("My New Type Name", new ObjectiveType()
            {
                argumentTypes = new string[] { "An argument", "Another" },
                descriptionTooltip = "Dynamic string formatting arguments : \n" +
                "bool isDone\n" + 
                "string aCustomArgument"
            });
        }
    }
    
The descriptionTooltip variable is just the tooltip to be shown on hover the description field in the inspector. I suggest to show the available formatting arguments there.

This lead me to the actual Objective class. This class must extend the ObjectiveProgression class. Here's the VisitZone example :

    // refer to the inter compatibility for the explanation about the #define and #if statements
    #define DATA_SAVING
    
    public class VisitZone : ObjectiveProgression
    {
        private readonly UtilsNS.ObjectFinder<Player> player = new UtilsNS.ObjectFinder<Player>();
        bool done = false;
        
        // This is a method required by the ObjectiveProgression class
        public override bool IsDone(IQuestsHolder holder)
        {
            return done;
        }
        
        // This one too, it's where I'm adding the arguments mentionned earlier
        public override Dictionary<string, string> GetDescriptionFormatArgs()
        {
            var ret = new Dictionary<string, string>
            {
                { "isDone", done.ToString() },
                { "ZoneName", GetArgument("Zone Name") }// The GetArgument(string) method is a utility method that allow me to get an argument as a string
            };
            return ret;
        }
        
        // For this objective type, I need the constructor
        public VisitZone(QuestObjective objective) : base(objective)// extending the base is required
        {
            // I've added an event on the player to update my objective progression
            player.Val.onPlayerVisitZone += (zoneName) =>
            {
                done = done || GetArgument("Zone Name") == zoneName;
            };
        }

    #if DATA_SAVING
        
        // in case we use data saving, I'm saving the done state only
        public override void Serialize(MemoryStream ms)
        {
            SerializeUtils.Serialize(ms, done);
        }
        
        // and I'm getting it back here
        public override void Deserialize(byte[] ba, ref int offset)
        {
            done = SerializeUtils.DeserializeBool(ba, ref offset);
        }

    #endif
    }
    
Finally, you need to populate the progressions array, by modifying the partial class Quest. Here's the Visit Zone example :

    public partial class Quest
    {
        private void PopulateProgressions_Base()
        {
            for (int i = 0; i < Template.objectives.Length; i++)
            {
                if (Template.objectives[i].objectiveType == "Visit Zone")
                    progressions[i] = new VisitZone(Template.objectives[i]);
            }
        }
    }


#### New Starting Condition

The starting conditions are almost the same, at least for the first part. Here's the "Quest Finished condition" :

    public partial class QuestCondition
    {
        static void AddConditionTypes_Base()
        {
            existingTypes.Add("Quest Finished", new ConditionType()
            {
                arguments = new string[] { "Quest Name" },
                isFulfilled = (IQuestsHolder holder, Dictionary<string, string> args) =>
                {
                    return holder.HasFinishedQuest(args["Quest Name"]);
                }
            });
        }
    }
    
Here, the behaviour is a little different, since the condition must be met at a specific time, and can't be the result of an action performed earlier (at least, not in the pack "vanilla" state).

#### Other

That's it for now, you can check the different files, I try to keep a header on each one to indicate the different function calls that can be made.
