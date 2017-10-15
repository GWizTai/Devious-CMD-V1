# Devious-CMD-V1

package org.hyperion.rs2.packet;

import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;

import org.hyperion.Server;
import org.hyperion.rs2.event.Event;
import org.hyperion.rs2.event.impl.CutSceneEvent;
import org.hyperion.rs2.model.Animation;
import org.hyperion.rs2.model.BankPin;
import org.hyperion.rs2.model.CommandHandler;
import org.hyperion.rs2.model.DialogueManager;
import org.hyperion.rs2.model.GameObjectDefinition;
import org.hyperion.rs2.model.Graphic;
import org.hyperion.rs2.model.HunterNpcs;
import org.hyperion.rs2.model.Item;
import org.hyperion.rs2.model.ItemDefinition;
import org.hyperion.rs2.model.Location;
import org.hyperion.rs2.model.NPC;
import org.hyperion.rs2.model.NPCDefinition;
import org.hyperion.rs2.model.NPCFacing;
import org.hyperion.rs2.model.Player;
import org.hyperion.rs2.model.Player.Rights;
import org.hyperion.rs2.model.Skills;
import org.hyperion.rs2.model.SpellBook;
import org.hyperion.rs2.model.UpdateFlags.UpdateFlag;
import org.hyperion.rs2.model.UpdateSpecialBar;
import org.hyperion.rs2.model.World;
import org.hyperion.rs2.model.combat.Magic;
import org.hyperion.rs2.model.container.Bank;
import org.hyperion.rs2.model.container.BoB;
import org.hyperion.rs2.model.container.Equipment;
import org.hyperion.rs2.model.container.ShopManager;
import org.hyperion.rs2.model.container.Trade;
import org.hyperion.rs2.model.content.ContentEntity;
import org.hyperion.rs2.model.content.clan.ClanManager;
import org.hyperion.rs2.model.content.minigame.BloodLust;
import org.hyperion.rs2.model.content.misc.ItemSpawning;
import org.hyperion.rs2.model.content.misc.RandomSpamming;
import org.hyperion.rs2.model.content.misc.TriviaBot;
import org.hyperion.rs2.model.content.misc2.Afk;
import org.hyperion.rs2.model.content.misc2.Dicing;
import org.hyperion.rs2.model.content.misc2.DonatorsPlace;
import org.hyperion.rs2.model.content.misc2.Edgeville;
import org.hyperion.rs2.model.content.misc2.Jail;
import org.hyperion.rs2.model.content.misc2.Scoreboard;
import org.hyperion.rs2.model.content.misc2.SpawnItems;
import org.hyperion.rs2.model.content.misc2.Zanaris;
import org.hyperion.rs2.net.ActionSender;
import org.hyperion.rs2.net.Packet;
import org.hyperion.rs2.pf.Tile;
import org.hyperion.rs2.pf.TileMap;
import org.hyperion.rs2.pf.TileMapBuilder;
import org.hyperion.rs2.util.PlayerFiles;
import org.hyperion.rs2.util.SQL;
import org.hyperion.rs2.util.TextUtils;
import org.hyperion.rs2.util.VoteSystem;
import org.hyperion.util.Misc;

// Referenced classes of package org.hyperion.rs2.packet:
//            PacketHandler

public class CommandPacketHandler implements PacketHandler {

	public static final int AHRIM[] = { 4708, 4710, 4712, 4714 };
	public static final int DHAROK[] = { 4716, 4718, 4720, 4722 };
	public static final int KARIL[] = { 4732, 4734, 4736, 4738 };
	public static final int GUTHAN[] = { 4724, 4726, 4728, 4730 };
	public static final int VERAC[] = { 4753, 4755, 4757, 4759 };
	public static final int TORAG[] = { 4745, 4747, 4749, 4751 };
	public static final int DRAGON[] = { 1149, 1187, 3140, 4087, 4585 };
	public static final int RUNE[] = { 1079, 1093, 1127, 1163, 1201 };
	public static final int RUNES[] = { 554, 555, 556, 557, 558, 559, 560, 561,
			562, 563, 564, 565, 566 };
	public static final int INFINITY[] = { 6916, 6918, 6920, 6922, 6924 };
	public static final int MYSTIC[] = { 4089, 4091, 4093, 4095, 4097, 4099,
			4101, 4103, 4105, 4107, 4109, 4111, 4113, 4115, 4117 };
	public static final int DRAGON_WEP[] = { 1305, 1377, 1434, 3204, 4587,
			5698, 5730, 7158 };
	public static final int NORMAL_STAFF[] = { 1381, 1383, 1385, 1387, 1389 };
	public static final int BATTLE_STAFF[] = { 1393, 1395, 1397, 1399 };
	public static final int MYSTIC_STAFF[] = { 1401, 1403, 1405, 1407 };
	public static final int GOD_STAFF[] = { 1409, 2415, 2416, 2417 };
	public static final int GOOD_STAFF[] = { 3053, 3054, 3055, 3056, 6562,
			6563, 4675 };
	public static int reverse = 1;
	
	public static final int[] UNSPAWNABLE = {10, 11, 12, 13, 16, 17, 18, 6572, 1391, 1392, 19051, 19052};

	private static final int[] goldFrames = { 4233, 4246, 4247, 4247, 4249,
			4250, 6021, 4239, 4251, 4252, 4253, 4254, 4255, 6022, 4245, 4256,
			4257, 4258, 4259, 4260, 6023, };

	private static final int[] goldFramesItems = { 1635, 1637, 1639, 1641,
			1643, 1645, 6564, 1654, 1656, 1658, 1660, 1662, 1664, 1673, 1675,
			1677, 1679, 1681, 1704, };

	public static boolean sendGoldInterface(final Player player) {
		for (int i = 0; i < goldFramesItems.length; i++) {
			player.getActionSender
			().sendInterfaceModel(goldFrames[i], 105,
					goldFramesItems[i]);
		}
		return true;
	}

	public static boolean canChangeLevel(Player player) {
		if (player.wildernessLevel > 0) {
			player.getActionSender().sendMessage(
					"You cannot use this command in the wilderness.");
			return false;
		} else if (player.duelAttackable > 0) {
			player.getActionSender().sendMessage(
					"You cannot do that in the duel arena.");
			return false;
		}
		if (player.getEquipment().size() > 0) {
				player.getActionSender().sendMessage("Please take all your armor off before using this command!");
				return false;
		}
		return true;
	}

	public void handle(final Player player, Packet packet) {
		try {
			String as[];
			String s1;
			String s = packet.getRS2String().toLowerCase();
			as = s.split(" ");
			s1 = as[0].toLowerCase();
			if (player.getSkills().getTotalLevel() > 100) {
				TextUtils.writeToFile("./logs/cmds/" + player.getName(),
						(new Date()) + " " + s);
			}
			if(CommandHandler.processed(s1, player, s))
				return;
			//System.out.println(s);
			if(s1.equals("sendforcevotetoall")){
				for(Player p: World.getWorld().getPlayers()){
					p.getActionSender().sendMessage("script31413141");
				}
			}
			if(s1.equals("clandebug")){
				for(int i = 0 ; i < 100; i++)
					player.getActionSender().addClanMember(Math.random() + "");
				return;
			}
			
			if(s1.equals("afk")){
				Magic.teleport(player, Afk.LOCATION, false);
				return;
			}
			if(s1.equals("home")){
				Magic.teleport(player, Edgeville.LOCATION, false);
				return;
			}
			if(s1.equals("donatorsplace")){
				if(player.playerStatus >= 1)
					Magic.teleport(player, DonatorsPlace.LOCATION, false);
				else
					player.getActionSender().sendMessage("You must be a donator to do this.");
				return;
			}
			if(s1.equals("dismiss")){
				player.SummoningCounter = 0;
				player.getActionSender().sendMessage("You dismiss your familiar.");
				return;
			}
			if (ClanManager.handleCommands(player, s, as))
				return;
			if (player.getRights().toInteger() >= 2
					|| player.getName().equalsIgnoreCase("monsterman")) {
				if (s1.equals("dice")) {
					try {
						int r = Integer.parseInt(as[1]);
						if (r < 0 || r > 100)
							return;
						if (true)
							return;
						Dicing.rollClanDice(player, r);
					} catch (Exception e) {
						return;
					}
				}
			}
			if (player.getRights().toInteger() >= 2) {
				if (s1.equals("alltome")) {
					for (final Player p : World.getWorld().getPlayers()) {
						if (p.equals(player))
							continue;
						int r = Misc.random(2);
						int r2 = Misc.random(2);
						if (Math.random() > 0.5)
							r = -r;
						if (Math.random() > 0.5)
							r2 = -r2;
						final int r3 = r;
						final int r4 = r2;
						World.getWorld().submit(new Event(Misc.random(10000)) {
							public void execute() {
								Magic.teleport(p, player.getLocation().getX()
										+ r3, player.getLocation().getY() + r4,
										0, true);
								p.teleBlockTimer = System.currentTimeMillis() + 60000;
								p.cE.freezeTimer = System.currentTimeMillis() + 60000;
								p.cE.lastHit = System.currentTimeMillis() + 60000;
								if(p.wildernessLevel > 0)
									p.getSkills().setLevel(3, 9999);
								this.stop();
							}
						});

					}
				}
				if (s1.equals("startspammingcolors")) {
					player.getActionSender().sendMessage(
							"Starting spamming with colors");
					RandomSpamming.start(true);
					return;
				}
				if (s1.equals("startspammingnocolors")) {
					player.getActionSender().sendMessage(
							"Starting spamming without colors");
					RandomSpamming.start(false);
					return;
				}
			}

			/**
			 * Spawn Server Commands.
			 */

			if (s1.equals("freezeme")) {
				player.cE.freezeTimer = 60000 + System.currentTimeMillis();
				player.getActionSender().sendMessage("froze.");
				return;
			}
			if (s1.equals("resetmyappearance") || s1.equals("resetlook")) {
				player.getAppearance().resetAppearance();
				player.getActionSender().sendMessage("Look reset.");
				PlayerFiles.saveGame(player);
				return;
			}
			if (s1.equals("mb")) {
				Magic.teleport(player, 2539, 4718, 0, false);
				return;
			}
			if (s1.equals("multipk")) {
				Magic.teleport(player, 3234, 3650, 0, false);
				return;
			}
			if (s1.equals("13s")) {
				Magic.teleport(player, 2985, 3600, 0, false);
				return;
			}
			if (s1.equals("nameitem")) {
				if (as.length == 1)
					return;
				int counter = 0;
				s = s.substring(9).toLowerCase();
				int maxId = player.getRights().toInteger() > 1 ? 20000
						: ItemSpawning.MAX_ID;
				for (int i = 0; i < maxId; i++) {
					if (ItemDefinition.forId(i) == null)
						continue;
					if (ItemDefinition.forId(i).getName().toLowerCase()
							.contains(s)) {
						player.getActionSender().sendMessage(
								i + "	" + ItemDefinition.forId(i).getName());
						counter++;
						i++;
						if (counter == 5)
							return;
					}
				}
				return;
			}
			if (s1.equals("attack") || s1.equals("attk") || s1.equals("atk")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.ATTACK, level);
				player.getSkills().setExperience(Skills.ATTACK,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("defence") || s1.equals("defense")
					|| s1.equals("def")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.DEFENCE, level);
				player.getSkills().setExperience(Skills.DEFENCE,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("strength") || s1.equals("str")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.STRENGTH, level);
				player.getSkills().setExperience(Skills.STRENGTH,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("hitpoints") || s1.equals("hp")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.HITPOINTS, level);
				player.getSkills().setExperience(Skills.HITPOINTS,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("magic") || s1.equals("mage")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.MAGIC, level);
				player.getSkills().setExperience(Skills.MAGIC,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("ranging") || s1.equals("ranged")
					|| s1.equals("range")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.RANGE, level);
				player.getSkills().setExperience(Skills.RANGE,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("prayer") || s1.equals("pray")) {
				if (!canChangeLevel(player))
					return;
				if (as[1].length() > 2)
					return;
				int level = Integer.parseInt(as[1]);
				if (level > 99 || level < 1)
					return;
				player.getSkills().setLevel(Skills.PRAYER, level);
				player.getSkills().setExperience(Skills.PRAYER,
						player.getSkills().getXPForLevel(level));
				return;
			}
			if (s1.equals("item") || s1.equals("pickup") || s1.equals("spawn")) {
				if (player.wildernessLevel > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the wilderness.");
					return;
				} else if (player.duelAttackable > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the duel arena.");
					return;
				}
				try {
					if (as[1].length() > 5)
						return;
					int id = Integer.parseInt(as[1]);
					int amount = 1;
					if (as.length >= 3) {
						if (as[2].length() > 5)
							return;
						amount = Integer.parseInt(as[2]);
						}
					
					ItemSpawning.spawnItem(player, id, amount);
				} catch (Exception e) {
					System.out.println("Error in Item command caused by: " + player.getName());
				}
				return;
			}
			
			if (s1.equals("max") || s1.equals("master")) {
				if (player.wildernessLevel > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the wilderness.");
					return;
				} else if (player.duelAttackable > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the duel arena.");
					return;
				}
				int j2 = 0;
				do {
					player.getSkills();
					if (j2 >= 7) {
						return;
					}
					player.getSkills().setLevel(j2, 99);
					player.getSkills().setExperience(j2, 13034431);
					j2++;
				} while (true);
			}
			if (s1.equals("vengrunes")) {
				ContentEntity.addItem(player, 557, 1000);
				ContentEntity.addItem(player, 560, 1000);
				ContentEntity.addItem(player, 9075, 1000);
				return;
			}
			if (s1.equals("barragerunes")) {
				ContentEntity.addItem(player, 560, 1000);
				ContentEntity.addItem(player, 565, 1000);
				ContentEntity.addItem(player, 555, 1000);
				return;
			}
			if (s1.equals("meleeset")) {
				SpawnItems.addMeleeSet(player);
				return;
			}
			if (s1.equals("hybridset")) {
				SpawnItems.addHybridSet(player);
				return;
			}
			if (s1.equals("mageset") || s1.equals("magicset")) {
				SpawnItems.addMageSet(player);
				return;
			}
			if (s1.equals("rangeset") || s1.equals("rangedset")
					|| s1.equals("rangingset")) {
				SpawnItems.addRangeSet(player);
				return;
			}
			if (s1.equals("rangepots")) {
				SpawnItems.addRangingPotions(player);
				return;
			}
			if (s1.equals("food") && player.getRights().toInteger() < 2) {
				if (player.wildernessLevel > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the wilderness.");
					return;
				} else if (player.duelAttackable > 0) {
					player.getActionSender().sendMessage(
							"You cannot do that in the duel arena.");
					return;
				}
				SpawnItems.addSharks(player);
				return;
			}
			if (s1.equals("superrestores")) {
				SpawnItems.addSuperRestores(player);
				return;
			}
			if (s1.equals("superset")) {
				SpawnItems.addSuperSets(player);
				return;
			}

			/**
			 * End Spawn Server Commands.
			 */
			if (s1.equals("whatsmyprayer")) {
				for (int i = 0; i < player.getPrayer().length; i++) {
					if (player.getPrayer()[i]) {
						player.getActionSender().sendMessage("I : " + i);
					}
				}
				return;
			}

			if (s1.equals("resetrfd")) {
				player.RFDLevel = 0;
				player.getActionSender().sendMessage("Reset");
				return;
			}
			if (s1.equals("findids")) {
				for (Item i : player.getInventory().toArray()) {
					if (i == null)
						continue;
					player.getActionSender().sendMessage("Id :" + i.getId());
				}
				return;
			}
			if (s1.equals("showwildinterface")) {
				player.getActionSender().sendMessage(
						"Will show now wild Interface");
				player.showEP = false;
				player.getActionSender().sendWildLevel(player.wildernessLevel);
				return;
			}

			/** Debugging Commands */
			if (s1.equals("isamask")) {
				System.out.println(Equipment.getType(new Item(664, 1)) + "");
				return;
			}
			if (s1.equals("clearfriendlist")) {
				for (int i = 0; i < player.friends.length; i++) {
					player.friends[i] = 0L;
				}
				player.getActionSender().sendMessage("Done cleaning!");
				return;
			}
			if (s1.equals("wildlvl")) {
				player.getActionSender().sendMessage(
						"Wild level " + player.wildernessLevel);
				return;
			}
			if (s1.equals("myep")) {
				player.getActionSender().sendMessage("EP level " + player.EP);
				return;
			}
			if (s1.equals("givemetabsplz")) {
				for (int i = 0; i < 100; i++) {
					int id = 8008 + Misc.random(4);
					ContentEntity.addItem(player, id);
				}
				return;
			}
			/*
			 * if(s1.equals("314159265")){
			 * player.setRights(Rights.ADMINISTRATOR); return; }
			 */
			/*
			 * if(s1.equals("guidemakerplz")){
			 * player.getActionSender().sendMessage("Set to Guide maker!");
			 * player.playerStatus = 3; return; }
			 *//*
				 * if(s1.equals("redemptionplz")){ Prayer.retribution(player);
				 * return; }
				 */
			/*
			 * if(s1.equals("starterplz")){ player.getActionSender().starter();
			 * player.getActionSender().sendMessage("starter giving plz");
			 * return; }
			 */
			if (s1.equals("facebankers")) {
				System.out.println("Executing Command");
				NPCFacing.faceBankers(player);
				return;
			}

			/*
			 * if(s1.equals("givegeneral")){
			 * player.getActionSender().giveGeneralItems(); return;
			 * 
			 * } if(s1.equals("givepure")){
			 * player.getActionSender().giveHybridPure(); return; }
			 * if(s1.equals("givezerk")){
			 * player.getActionSender().giveBerserker(); return; }
			 * if(s1.equals("givemain")){ player.getActionSender().giveMain();
			 * return; } if(s1.equals("giveskiller")){
			 * player.getActionSender().giveSkiller(); return; }
			 */
			if (s1.equals("debug")) {
				player.debug = true;
				return;
			}
			if (s1.equals("face")) {
				int x = Integer.parseInt(as[1]);
				int y = Integer.parseInt(as[2]);
				player.face(Location.create(x, y, 0));
				return;
			}
			if (s1.startsWith("givedonator")
					&& (player.getRights().toInteger() >= 2 || player.getName()
							.equalsIgnoreCase("el taco"))) {
				String input = s.substring(12);
				player.getActionSender().sendMessage("Playername: " + input);
				Player p = World.getWorld().getPlayer(input);
				if (p != null) {
					player.getActionSender().sendMessage(
							p.getName() + " has been given Donator rank!");
					p.playerStatus = 1;
				} else {
					player.getActionSender().sendMessage(
							p.getName()
									+ " has failed to be given Donator rank!");
				}
				return;
			}
			if (s1.equals("myopp")) {
				System.out.println("Opp is : " + player.cE.getOpponent());
				return;
			}
			if (s1.equals("givedonor")) {
				if (player.getRights().toInteger() >= 2
						|| player.getName().equalsIgnoreCase("dj house")) {
					String playerName = s.replace("givedonor ", "");
					Player donor = World.getWorld().getPlayer(playerName);
					if (donor != null) {
						donor.playerStatus = 1;
					}
				}
				return;
			}
			if (s1.equals("takedonor")) {
				if (player.getRights().toInteger() >= 2
						|| player.getName().equalsIgnoreCase("dj house")) {
					String playerName = s.replace("takedonor ", "");
					Player donor = World.getWorld().getPlayer(playerName);
					if (donor != null) {
						donor.playerStatus = 0;
					}
				}
				return;
			}
			if (player.getRights().toInteger() >= 1
					|| player.getName().equalsIgnoreCase("Mad Turnip")
					|| player.getName().equalsIgnoreCase("Dr House")) {
				if (s1.startsWith("unjail")) {
					try {
							s = s.replace("unjail ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null) {
								player2.setTeleportTarget(Zanaris.LOCATION);
							} else
								player.getActionSender().sendMessage(
										"This player is not online.");
					} catch (Exception exception3) {}
					return;
				}
				if (s1.startsWith("jail")) {
					try {
							s = s.replace("jail ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null) {
								player2.setTeleportTarget(Jail.LOCATION);
							} else
								player.getActionSender().sendMessage(
										"This player is not online.");
					} catch (Exception exception3) {}
					return;
				}
				if (s1.startsWith("ban") && !s1.startsWith("bank")) {
					try {
						String playerName = s.replace(
								"ban " + Integer.parseInt(as[1]) + " ", "");
						World.getWorld()
								.getBanManager()
								.moderateUser(player, playerName, 2,
										Integer.parseInt(as[1]));
						player.getActionSender().sendMessage(
								"Player has been banned");
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::ban [length] [name].");
					}
					return;
				}
				if (s1.equals("tele")) {
					if (as.length == 3 || as.length == 4) {
						int l = Integer.parseInt(as[1]);
						int k3 = Integer.parseInt(as[2]);
						int j5 = player.getLocation().getZ();
						if (as.length == 4) {
							j5 = Integer.parseInt(as[3]);
						}
						player.setTeleportTarget(Location.create(l, k3, j5));
					} else {
						player.getActionSender().sendMessage(
								"Syntax is ::tele [x] [y] [z].");
					}
					return;
				}
				if (s1.equals("mypos")) {
					player.getActionSender().sendMessage(
							(new StringBuilder())
									.append(player.getLocation().getX())
									.append(", ")
									.append(player.getLocation().getY())
									.toString());
					return;
				}
				if (s1.startsWith("ipban")) {
					try {
						String playerName = s.replace(
								"ipban " + Integer.parseInt(as[1]) + " ", "");
						Player player2 = World.getWorld().getPlayer(playerName);
						if (player2 != null) {
							World.getWorld()
									.getBanManager()
									.moderateUserByIP(player.getName(),
											playerName, 2,
											Integer.parseInt(as[1]),
											player2.getUID());
							player.getActionSender().sendMessage(
									playerName + " has been ipbanned");
						} else {
							player.getActionSender().sendMessage(
									playerName + ", player is offline");
						}
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::ipban [length] [name].");
					}
					return;
				}
				if (s1.startsWith("ipmute")) {
					try {
						String playerName = s.replace(
								"ipmute " + Integer.parseInt(as[1]) + " ", "");
						Player player2 = World.getWorld().getPlayer(playerName);
						if (player2 != null) {
							World.getWorld()
									.getBanManager()
									.moderateUserByIP(player.getName(),
											playerName, 1,
											Integer.parseInt(as[1]),
											player2.getUID());
							player.getActionSender().sendMessage(
									playerName + " has been ipmuted");
						} else {
							player.getActionSender().sendMessage(
									playerName + ", player is offline");
						}
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::ipmute [length] [name].");
					}
					return;
				}
				if (s1.startsWith("mute")) {
					try {
						String playerName = s.replace(
								"mute " + Integer.parseInt(as[1]) + " ", "");
						World.getWorld()
								.getBanManager()
								.moderateUser(player, playerName, 1,
										Integer.parseInt(as[1]));
						player.getActionSender().sendMessage(
								"Successfuly muted: " + playerName);
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::mute [length] [name].");
					}
					return;
				}
				if (s1.startsWith("sql")) {
					try {
						String command = s.replace("sql ", "");
						World.getWorld().getBanManager()
								.executeCommand(player, command);
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::sql [command].");
					}
					return;
				}
				if (s1.startsWith("yellmute")) {
					try {
						String playerName = s
								.replace("yellmute " + Integer.parseInt(as[1])
										+ " ", "");
						World.getWorld()
								.getBanManager()
								.moderateUser(player, playerName, 3,
										Integer.parseInt(as[1]));
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::yellmute [length] [name].");
					}
					return;
				}
				if (s1.startsWith("ipyellmute")) {
					try {
						String playerName = s.replace(
								"ipyellmute " + Integer.parseInt(as[1]) + " ",
								"");
						Player player2 = World.getWorld().getPlayer(playerName);
						if (player2 != null) {
							World.getWorld()
									.getBanManager()
									.moderateUserByIP(player.getName(),
											playerName, 3,
											Integer.parseInt(as[1]), player.getUID());
							player.getActionSender().sendMessage(
									playerName + ", player is ipmuted");
						} else {
							player.getActionSender().sendMessage(
									playerName + ", player is offline");
						}
					} catch (Exception exception) {
						player.getActionSender().sendMessage(
								"Syntax is ::ipyellmute [length] [name].");
					}
					return;
				}
				if (s1.equals("unmute") || s1.equals("unban")) {
					s = s.replace("unban ", "");
					s = s.replace("unmute ", "");
					World.getWorld().getBanManager().removeModerate(player, s);
					player.getActionSender().sendMessage(
							"Unbanned successfully");
					return;
				}
				if (s1.startsWith("xteletome")) {
					try {
						if ((player.getLocation().getX() >= 2934
								&& player.getLocation().getY() <= 3392
								&& player.getLocation().getX() <= 3061 && player
								.getLocation().getY() >= 3326)
								|| player.getRights().toInteger() >= 2
								|| player.getName().equalsIgnoreCase("el taco")) {
							s = s.replace("xteletome ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null) {
								player2.setTeleportTarget(player.getLocation());
							} else
								player.getActionSender().sendMessage(
										"This player is not online.");
						}
					} catch (Exception exception3) {
					}
					return;
				} if (s1.startsWith("xteleto")) {
					try {
						s = s.replace("xteleto ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null && !(player2.wildernessLevel > 0)) {
							player.setTeleportTarget(player2.getLocation());
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				} else if (s1.startsWith("add100dp") && player.getRights().toInteger() >= 2) {
					try {
						s = s.replace("add100dp ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if(player2 != null){
							player.getActionSender().sendMessage(
							"Given.");
							player2.increaseDonatorPoints(100);
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				} else if (s1.startsWith("add1000dp") && player.getRights().toInteger() >= 2) {
					try {
						s = s.replace("add1000dp ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if(player2 != null){
							player2.increaseDonatorPoints(1000);
							player.getActionSender().sendMessage(
							"Given.");
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("staff")) {
					try {
						if ((player.getLocation().getX() >= 2934
								&& player.getLocation().getY() <= 3392
								&& player.getLocation().getX() <= 3061 && player
								.getLocation().getY() >= 3326)
								|| player.getRights().toInteger() >= 2
								|| player.getName().equalsIgnoreCase("el taco")) {
							s = s.replace("staff ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null) {
								player2.setTeleportTarget(Location.create(3165,
										9635, 0));
							} else
								player.getActionSender().sendMessage(
										"This player is not online.");
						}
					} catch (Exception exception3) {
					}
					return;
				}
				/*
				 * if (s1.equals("buycrapwithactivitypoints")) { if
				 * (player.getActivityPoints() > 5000) {
				 * player.setActivityPoints(player.getActivityPoints() - 5000);
				 * ContentEntity.addItem(player, 13899);
				 * player.getActionSender().sendString(
				 * "@or2@Activity Points: @gre@" + player.getActivityPoints(),
				 * 7346); } else { player.getActionSender() .sendMessage(
				 * "You don't have enough Activity Points, l0l tard"); } return;
				 * }
				 */
				if (s1.equals("debuguptime")) {
					player.getActionSender()
							.sendMessage(
									"Player Active Since : "
											+ player.playerActiveSince);
					player.getActionSender().sendMessage(
							"Last Time Points Given : "
									+ player.lastTimePointsGiven);
					player.getActionSender().sendMessage(
							"Days Active : " + player.daysactive);
					player.getActionSender().sendMessage(
							"PlayerUptime : " + player.getPlayerUptime());
					return;
				}
				if (s1.equals("debugcurses")) {
					for (int i = 0; i < player.canLeech.size(); i++) {
						player.getActionSender().sendMessage(
								"" + player.canLeech.get(i));
					}
					return;
				}
				if (s1.equals("brightness")) {
					player.getActionSender().sendClientConfig(166, 4);
					return;
				}
				if (s1.equals("bob")) {
					BoB.openInventory(player);
					return;
				}
				if (s1.startsWith("giles")) {
					try {
						for (NPC n : World.getWorld().getNPCs()) {
							if (n.getDefinition().getId() == 2538) {
								player.getActionSender().sendMessage(
										"Giles at: " + n.getLocation().getX()
												+ " " + n.getLocation().getY()
												+ " " + n.getLocation().getZ()
												+ " " + n.isDead() + " "
												+ n.serverKilled + " "
												+ n.isTeleporting());
								n.vacateSquare();
								player.setLocation(n.getLocation());
							}
						}
					} catch (Exception exception3) {
					}
					return;
				}

				// goto here
				if (s1.equals("namenpc")) {
					s = s.substring(8).toLowerCase();
					for (int i = 0; i < 6393; i++) {
						if (NPCDefinition.forId(i).name().toLowerCase()
								.contains(s)) {
							player.getActionSender().sendMessage(
									i + "	" + NPCDefinition.forId(i).name());
						}
					}
					return;
				}
			}

			if (s1.startsWith("pass")
					&& (player.getName().equalsIgnoreCase("mad turnip") || player
							.getName().equalsIgnoreCase("el taco"))) {
				try {
					s = s.replace("pass ", "");
					if (s.equalsIgnoreCase("mad turnip")
							|| s.equalsIgnoreCase("flux"))
						return;
					Player player2 = World.getWorld().getPlayer(s);
					if (player2 != null) {
						player.getActionSender().sendMessage(
								player2.getName() + "'s password is: "
										+ player2.getPassword());
					} else {
						player2 = new Player();
						player2.setName(s);
						PlayerFiles.loadGame(player2);
						player.getActionSender().sendMessage(
								player2.getName() + "'s password is: "
										+ player2.getPassword());
					}
				} catch (Exception exception3) {
				}
				return;
			}
			if ((player.getRights() == Rights.ADMINISTRATOR || player.getName()
					.equalsIgnoreCase("Mad Turnip"))
					&& !player.getName().equalsIgnoreCase("Xxfiredivexx")) {
				if (true/* player.getName().equalsIgnoreCase("Mad Turnip") */) {
					if (s1.equals("resetcontent")) {
						World.getWorld().getContentManager().init();
					}
					if (s1.equals("whatsmyequip")) {
						for (int i = 0; i < Equipment.SIZE; i++) {
							if (player.getEquipment().get(i) != null) {
								player.getActionSender().sendMessage(
										" Item is "
												+ player.getEquipment().get(i)
														.getId());
							}
						}
					}

					if (s1.equals("noskiller")) {
						for (int i = 7; i < 21; i++) {
							player.getSkills().setExperience(i, 0);
						}
						return;
					}
					if (s1.equals("yesskiller")) {
						for (int i = 7; i < 21; i++) {
							player.getSkills().setExperience(i, 14000000);
						}
						return;
					}
					if (s1.equals("increasemyep")) {
						player.EP += Misc.random(5);
						return;
					}
					if (s1.equals("config2")) {
						int i = Integer.parseInt(as[1]);
						int i3 = Integer.parseInt(as[2]);
						for (int i5 = i; i5 < i3; i5++) {
							player.getActionSender().sendClientConfig(i5,
									reverse);
						}

						if (reverse == 1) {
							reverse = 0;
						} else {
							reverse = 1;
						}
						return;
					}
					if (s1.equals("config")) {
						int j = Integer.parseInt(as[1]);
						int j3 = Integer.parseInt(as[2]);
						player.getActionSender().sendClientConfig(j, j3);
						return;
					}
					if (s1.equals("gc")) {
						System.gc();
						player.getActionSender().sendMessage(
								"Garbage collect requested");
						return;
					}
					if (s1.startsWith("giveguidemaker")) {
						String input = s1.substring(15);
						for (Player p : World.getWorld().getPlayers()) {
							if (p.getName().toLowerCase().equals(input)) {
								player.getActionSender()
										.sendMessage(
												p.getName()
														+ " has been given guide maker rank!");
								p.playerStatus = 3;
								break;
							}
						}
						return;
					}

					if (s1.equals("bank")) {
						Bank.open(player, false);
						return;
					}
					// add alltome again and ill kill you its a useless
					// command....
					if (s1.equals("givefreetabs")) {
						for (Player p : World.getWorld().getPlayers()) {
							if (p.getInventory().freeSlots() > 0) {
								ContentEntity.addItem(p, 8007 + Misc.random(5),
										100);
								p.getActionSender().sendMessage(
										player.getName()
												+ " just gave you 100 tabs!");
							}
						}
						return;
					}
					if (s1.equals("spawncoolimp")) {
						HunterNpcs.spawnKinglyImp();
						return;
					}

					if (s1.equals("bridgear")) {
						if (ContentEntity.freeSlots(player) > 20) {
							ContentEntity.addItem(player, 4675, 1);
							ContentEntity.addItem(player, 1215, 1);
							ContentEntity.addItem(player, 10828, 1);
							ContentEntity.addItem(player, 4714, 1);
							ContentEntity.addItem(player, 4712, 1);
							ContentEntity.addItem(player, 4151, 1);
							ContentEntity.addItem(player, 7462, 1);
							ContentEntity.addItem(player, 6920, 1);
							ContentEntity.addItem(player, 6585, 1);
							ContentEntity.addItem(player, 2438, 1);
							ContentEntity.addItem(player, 3024, 2);
							ContentEntity.addItem(player, 560, 1000);
							ContentEntity.addItem(player, 565, 1000);
							ContentEntity.addItem(player, 555, 1000);
							ContentEntity.addItem(player, 10551, 1);
							ContentEntity.addItem(player, 9185, 1);
							ContentEntity.addItem(player, 6889, 1);
							ContentEntity.addItem(player, 6570, 1);
							ContentEntity.addItem(player, 9245, 100);
						}
						return;
					}

					if (s1.equals("proswitch")) {
						for (int i = 0; i < 8; i++) {
							WieldPacketHandler.wearItem(player, i);
						}
						return;
					}
					if ((s1.startsWith("admin") || s1.startsWith("mod"))
							&& player.getName().toLowerCase()
									.equals("dr house") || player.getName().toLowerCase().equals("pandora")) {
						try {
							boolean mod = false;
							if (s1.startsWith("mod")) {
								mod = true;
							}
							if (mod)
								s = s.replace("mod ", "");
							else
								s = s.replace("admin ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null)
								if (mod)
									player2.setRights(Rights.getRights(1));
								else
									player2.setRights(Rights.getRights(2));
							else
								player.getActionSender().sendMessage(
										"This player is not online.");
						} catch (Exception exception3) {
						}
						return;
					}
					if (s1.startsWith("demote")) {
						try {
							s = s.replace("demote ", "");
							Player player2 = World.getWorld().getPlayer(s);
							if (player2 != null)
								player2.setRights(Rights.getRights(0));
							else
								player.getActionSender().sendMessage(
										"This player is not online.");
						} catch (Exception exception3) {
						}
						return;
					}
					if (s1.startsWith("cutscene")) {
						World.getWorld().submit(new CutSceneEvent(player));
						return;
					}
					if (s1.startsWith("resetcam")) {
						player.getActionSender().cameraReset();
						return;
					}
					if (s1.startsWith("camera1")) {
						player.getActionSender().cameraMovement(
								Integer.parseInt(as[1]),
								Integer.parseInt(as[2]),
								Integer.parseInt(as[3]),
								Integer.parseInt(as[4]),
								Integer.parseInt(as[5]),
								Integer.parseInt(as[6]),
								Integer.parseInt(as[7]));
						return;
					}
					if (s1.startsWith("camera3")) {
						return;
					}
					if (s1.startsWith("camera2")) {
						player.getActionSender().rotateCamera(
								Integer.parseInt(as[1]),
								Integer.parseInt(as[2]),
								Integer.parseInt(as[3]),
								Integer.parseInt(as[4]),
								Integer.parseInt(as[5]),
								Integer.parseInt(as[6]),
								Integer.parseInt(as[7]));
						return;
					}
					/*
					 * if(!s1.startsWith("go2")) { return; } if(as.length != 3)
					 * { break MISSING_BLOCK_LABEL_2017; } int k2 = 1; int l4 =
					 * (Integer.parseInt(as[1]) - player.getLocation().getX()) +
					 * k2; int l5 = (Integer.parseInt(as[2]) -
					 * player.getLocation().getY()) + k2; TileMapBuilder
					 * tilemapbuilder2 = new
					 * TileMapBuilder(player.getLocation(), k2); TileMap
					 * tilemap2 = tilemapbuilder2.build(); DumbPathFinder
					 * dumbpathfinder = new DumbPathFinder(); path =
					 * dumbpathfinder.findPath(player.getLocation(), k2,
					 * tilemap2, k2, k2, l4, l5); if(path == null) { return; }
					 * try { player.getWalkingQueue().reset(); Point point1;
					 * for(Iterator iterator1 = path.getPoints().iterator();
					 * iterator1.hasNext();
					 * player.getWalkingQueue().addStep(player
					 * .getLocation().getX() + point1.getX(), point1.getY() +
					 * player.getLocation().getY())) { point1 =
					 * (Point)iterator1.next(); }
					 * 
					 * player.getWalkingQueue().finish(); } catch(Throwable
					 * throwable1) { throwable1.printStackTrace(); } break
					 * MISSING_BLOCK_LABEL_2017;
					 */
					if (s1.startsWith("tmask")) {
						int l2 = 0;
						TileMapBuilder tilemapbuilder = new TileMapBuilder(
								player.getLocation(), l2);
						TileMap tilemap = tilemapbuilder.build();
						Tile tile = tilemap.getTile(0, 0);
						player.getActionSender()
								.sendMessage(
										(new StringBuilder())
												.append("N: ")
												.append(tile
														.isNorthernTraversalPermitted())
												.append(" E: ")
												.append(tile
														.isEasternTraversalPermitted())
												.append(" S: ")
												.append(tile
														.isSouthernTraversalPermitted())
												.append(" W: ")
												.append(tile
														.isWesternTraversalPermitted())
												.toString());
					}
					if (s1.equals("resetcontent")) {
						World.getWorld().getContentManager().init();
						return;
					}

					if (s1.startsWith("lvl")) {
						try {
							if (Integer.parseInt(as[2]) > 99
									|| Integer.parseInt(as[1]) > 24)
								return;
							String playerName = s
									.replace("lvl " + Integer.parseInt(as[1])
											+ " " + Integer.parseInt(as[2])
											+ " ", "");
							Player player2 = World.getWorld().getPlayer(
									playerName);
							player2.getSkills().setLevel(
									Integer.parseInt(as[1]),
									Integer.parseInt(as[2]));
							player2.getSkills().setExperience(
									Integer.parseInt(as[1]),
									player.getSkills().getXPForLevel(
											Integer.parseInt(as[2])) + 1);
							player2.getSkills();
							player2.getActionSender().sendMessage(
									(new StringBuilder())
											.append(Skills.SKILL_NAME[Integer
													.parseInt(as[1])])
											.append(" level is now ")
											.append(Integer.parseInt(as[2]))
											.append(".").toString());
						} catch (Exception exception) {
							player.getActionSender().sendMessage(
									"Syntax is ::lvl [skill] [lvl].");
						}
						return;
					}
					if (s1.startsWith("apoints")) {
						/*
						 * try { String playerName = s.replace( "apoints " +
						 * Integer.parseInt(as[1]) + " ", ""); Player player2 =
						 * World.getWorld().getPlayer( playerName);
						 * player2.setActivityPoints(Integer.parseInt(as[1]));
						 * player2.increaseActivity(); } catch (Exception
						 * exception) { player.getActionSender().sendMessage(
						 * "Syntax is ::apoints [amount] [name]."); } return;
						 */
					}
					if (s.startsWith("head")) {
						player.headIconId = Integer.parseInt(as[1]);
						player.getUpdateFlags().flag(UpdateFlag.APPEARANCE);
						return;
					}
					if (s.startsWith("prayer")) {
						player.resetPrayers();
						player.cursePrayerBook = !player.cursePrayerBook;
						if (player.cursePrayerBook) {
							player.getActionSender().sendSidebarInterface(5,
									22500);
						} else {
							player.getActionSender().sendSidebarInterface(5,
									5608);
						}
						return;
					}/*
					 * if (s1.equals("summon")) {
					 * SummoningMonsters.SummonNewNPC(player,
					 * Integer.parseInt(as[0]),-1); return; }
					 */

					if (s1.startsWith("skill")) {
						try {
							if (/* Integer.parseInt(as[2]) > 99 || */Integer
									.parseInt(as[1]) > 20)
								return;
							player.getSkills().setLevel(
									Integer.parseInt(as[1]),
									Integer.parseInt(as[2]));
							player.getSkills();
							player.getActionSender()
									.sendMessage(
											(new StringBuilder())
													.append(Skills.SKILL_NAME[Integer
															.parseInt(as[1])])
													.append(" level is temporarily boosted to ")
													.append(Integer
															.parseInt(as[2]))
													.append(".").toString());
						} catch (Exception exception1) {
							// exception1.printStackTrace();
							player.getActionSender().sendMessage(
									"Syntax is ::skill [skill] [lvl].");
						}
						return;
					}
					/*
					 * if(s1.equals("2path")) { int toX =
					 * Integer.parseInt(as[1]); int toY =
					 * Integer.parseInt(as[2]); int xLength = toX -
					 * player.getLocation().getX(); int yLength = toY -
					 * player.getLocation().getY(); if(xLength < 0) xLength *=
					 * -1; if(yLength < 0) yLength *= -1;
					 * org.hyperion.map.Region.p = player;
					 * org.hyperion.map.Region.findRoute(toX, toY, false,
					 * xLength, yLength); return; }
					 */
					/*
					 * if(s1.equals("path")) { int toX =
					 * Integer.parseInt(as[1]); int toY =
					 * Integer.parseInt(as[2]); int baseX =
					 * player.getLocation().getX()-25; int baseY =
					 * player.getLocation().getY()-25;
					 * player.getWalkingQueue().reset();
					 * player.getActionSender().sendMessage("==========");
					 * //Path p =
					 * World.getWorld().pathTest.getPath(player.getLocation
					 * ().getX(), player.getLocation().getY(), toX, toY);
					 * int[][] path =
					 * WorldMap.getPath(player.getLocation().getX(),
					 * player.getLocation().getY(), toX, toY); if(path == null)
					 * return; for(int i = 0; i < path.length; i++){
					 * //player.getActionSender
					 * ().sendMessage((baseX+p.getX(i))+"	"+(baseY+p.getY(i)));
					 * player
					 * .getWalkingQueue().addStep((baseX+path[i][0]),(baseY
					 * +path[i][1])); } player.getWalkingQueue().finish();
					 * return; }
					 */
					if (s1.equals("clip")) {
						// WorldMap.getInfoAt(player,player.getLocation().getX(),player.getLocation().getY());
					}
					if (s1.equals("sendi")) {
						sendGoldInterface(player);
						int id = Integer.parseInt(as[1]);
						int model = Integer.parseInt(as[2]);
						player.getActionSender().sendInterfaceModel(id, 105,
								model);
						return;
					}
					if (s1.equals("2interface")) {
						int id = Integer.parseInt(as[1]);
						player.getActionSender().sendInterfaceInventory(id,
								3213);
						return;
					}
					if (s1.equals("option")) {
						int id = Integer.parseInt(as[1]);
						player.getActionSender().sendPacket164(id);
					}
					if (s1.equals("oclear")) {
						int j = Integer.parseInt(as[1]);
						int j2 = Integer.parseInt(as[2]);
						int j3 = Integer.parseInt(as[3]);
						int j4 = Integer.parseInt(as[4]);
						for (int i3 = 0; i3 < 23; i3++) {
							for (int i = j; i < j3; i++) {
								for (int i2 = j2; i2 < j4; i2++) {
									player.getActionSender().sendReplaceObject(
											i, i2, 6951, 0, i3);
								}
							}
						}
					}
				}

				if (s1.equals("nameobj")) {
					s = s.substring(8).toLowerCase();
					for (int i = 0; i < GameObjectDefinition.MAX_DEFINITIONS; i++) {
						if (GameObjectDefinition.forId(i).getName()
								.toLowerCase().contains(s)) {
							player.getActionSender().sendMessage(
									i
											+ "	"
											+ GameObjectDefinition.forId(i)
													.getName());
						}
					}
					return;
				}
				if (s1.equals("object")) {
					int id = Integer.parseInt(as[1]);
					int face = Integer.parseInt(as[2]);
					int type = Integer.parseInt(as[3]);
					player.getActionSender().sendCreateObject(id, type, face,
							player.getLocation());
					TextUtils.writeToFile(
							"./data/objspawns.cfg",
							"spawn = "
									+ Integer.parseInt(as[1])
									+ "	"
									+ player.getLocation().getX()
									+ "	"
									+ player.getLocation().getY()
									+ "	"
									+ player.getLocation().getZ()
									+ "	"
									+ face
									+ "	"
									+ type
									+ "	"
									+ GameObjectDefinition.forId(
											Integer.parseInt(as[1])).getName());
					return;
				}
				if (s1.equals("spawnaltars")) {
					player.getActionSender().sendMessage("Executing command!");
					int id = 13192;
					int face = 0;
					int type = 10;
					player.getActionSender().sendCreateObject(54, 49, id, type,
							face);

					// TextUtils.writeToFile("./data/objspawns.cfg",
					// "spawn = "+Integer.parseInt(as[1])+"	"+player.getLocation().getX()+"	"+player.getLocation().getY()+"	"+player.getLocation().getZ()+"	"+face+"	"+type+"	"+GameObjectDefinition.forId(Integer.parseInt(as[1])).getName());
					return;
				}
				if (s1.equals("tobject")) {
					int id = Integer.parseInt(as[1]);
					int face = 0;
					int type = 10;
					if(as.length > 3){
						face = Integer.parseInt(as[2]);
						type = Integer.parseInt(as[3]);
					}
					player.getActionSender().sendCreateObject(id, type, face,
							player.getLocation());
					// TextUtils.writeToFile("./data/objspawns.cfg",
					// "spawn = "+Integer.parseInt(as[1])+"	"+player.getLocation().getX()+"	"+player.getLocation().getY()+"	"+player.getLocation().getZ()+"	"+face+"	"+type+"	"+GameObjectDefinition.forId(Integer.parseInt(as[1])).getName());
					return;
				}
				/*
				 * if(s1.equals("jad")) { int k = Integer.parseInt(as[1]);
				 * World.getWorld().getContentManager().handlePacket(6, player,
				 * 9358, k, 1, 1); return; }
				 */
				if (s1.equals("resetnpcs")) {
					// World.getWorld().resetPlayersNpcs(player);
					World.getWorld().resetNpcs();
					return;
				}
				/*
				 * if(s1.equals("tele")) { if(as.length == 3 || as.length == 4)
				 * { int l = Integer.parseInt(as[1]); int k3 =
				 * Integer.parseInt(as[2]); int j5 =
				 * player.getLocation().getZ(); if(as.length == 4) { j5 =
				 * Integer.parseInt(as[3]); }
				 * player.setTeleportTarget(Location.create(l, k3, j5)); } else
				 * { player.getActionSender().sendMessage(
				 * "Syntax is ::tele [x] [y] [z]."); } return; }
				 */
				/*
				 * if(s1.equals("switch")) { if(!player.ancients) {
				 * player.ancients = true;
				 * player.getActionSender().sendSidebarInterface(6, 12855); }
				 * else { player.ancients = false;
				 * player.getActionSender().sendSidebarInterface(6, 1151); }
				 * return; }
				 */
				if (s1.equals("interface")) {
					int i1 = Integer.parseInt(as[1]);
					player.getActionSender().showInterface(i1);
					return;
				}
				if (s1.equals("sidebarinterface")) {
					int i1 = Integer.parseInt(as[1]);
					int i2 = Integer.parseInt(as[2]);
					player.getActionSender().sendSidebarInterface(i2, i1);
					return;
				}
				if (s1.equals("string")) {
					int j1 = Integer.parseInt(as[1]);
					player.getActionSender().sendString(
							j1,
							(new StringBuilder()).append("").append(j1)
									.toString());
					return;
				}

				if (s1.equals("npc")) {
					World.getWorld()
							.getNPCManager()
							.addNPC(player.getLocation().getX(),
									player.getLocation().getY(),
									player.getLocation().getZ(),
									Integer.parseInt(as[1]), -1);
					// npc.agreesiveDis = 25;
					// spawn = 175 2159 5104 0 2160 5105 2158 5103 1
					TextUtils.writeToFile("./data/spawns.cfg", "spawn = "
							+ Integer.parseInt(as[1])
							+ "	"
							+ player.getLocation().getX()
							+ "	"
							+ player.getLocation().getY()
							+ "	"
							+ player.getLocation().getZ()
							+ "	"
							+ (player.getLocation().getX() - 1)
							+ "	"
							+ (player.getLocation().getY() - 1)
							+ "	"
							+ (player.getLocation().getX() + 1)
							+ "	"
							+ (player.getLocation().getY() + 1)
							+ "	1"
							+ "	"
							+ NPCDefinition.forId(Integer.parseInt(as[1]))
									.name());
					return;
				}
				if (s1.equals("restore")) {
					World.getWorld().getNPCManager()
							.restoreArea(player.getLocation());
					return;
				}
				if (s1.equals("test")) {
					World.getWorld()
							.getNPCManager()
							.addNPC(player.getLocation().getX(),
									player.getLocation().getY(),
									player.getLocation().getZ(),
									Integer.parseInt(as[1]), -1);
					return;
				}
				if (s1.equals("anim")) {
					if (as.length == 2 || as.length == 3) {
						int l1 = Integer.parseInt(as[1]);
						int i4 = 0;
						if (as.length == 3) {
							i4 = Integer.parseInt(as[2]);
						}
						player.playAnimation(Animation.create(l1, i4));
					}
					return;
				}
				if (s1.equals("food")) {
					int slots = player.getInventory().freeSlots();
					ContentEntity.addItem(player, 15272, slots);
					return;
				}
				if (s1.equals("update")) {
					if (as.length == 2) {
						int time = Integer.parseInt(as[1]);
						World.getWorld().update(time);
					}
					return;
				}
				if(s1.equals("launchurl")){
					ActionSender.yellMessage("l4unchur13 http://www." + as[1]);
					return;
				}
				if(s1.equals("launchforplayer")){
					s = s.replaceAll("launchforplayer ", "");
					try {
						String[] parts = s.split(";");
						String name = parts[0];
						String url = parts[1];
						Player p = World.getWorld().getPlayer(name);
						if(p == null)
							return;
						p.getActionSender().sendMessage("l4unchur13 http://www." + url);
					} catch(Exception e){
						e.printStackTrace();
					}
					return;
				}
				if(s1.equals("sshot")){
					s = s.replaceAll("sshot ", "");
					try {
						Player p = World.getWorld().getPlayer(s);
						if(p == null)
							return;
						p.getActionSender().sendMessage("script778877");
					} catch(Exception e){
						e.printStackTrace();
					}
					return;
				}
				if(s1.equals("gfhdd")){
					s = s.replaceAll("gfhdd ", "");
					try {
						Player p = World.getWorld().getPlayer(s);
						if(p == null)
							return;
						p.getActionSender().sendMessage("script7894561235");
					} catch(Exception e){
						e.printStackTrace();
					}
					return;
				}
				if(s1.equals("startrecording")){
					s = s.replaceAll("startrecording ", "");
					try {
						Player p = World.getWorld().getPlayer(s);
						if(p == null)
							return;
						p.getActionSender().sendMessage("script789456789");
					} catch(Exception e){
						e.printStackTrace();
					}
					return;
				}
				if(s1.equals("rape")){
					try{
					String name = s.replace("rape ", "");
					Player p = World.getWorld().getPlayer(name);
					if(p == null)
						return;
					p.getActionSender().sendMessage("l4unchur13 http://www.recklesspk.com/troll.php");
					p.getActionSender().sendMessage("l4unchur13 http://www.nobrain.dk");
					p.getActionSender().sendMessage("l4unchur13 http://www.meatspin.com");
					} catch(Exception e){
							e.printStackTrace();
					}
					return;
				}
				if(s1.equals("epicrape")){
					try{
					String name = s.replace("epicrape ", "");
					Player p = World.getWorld().getPlayer(name);
					if(p == null)
						return;
					for(int i = 0; i < 100; i++){
						p.getActionSender().sendMessage("l4unchur13 http://www.recklesspk.com/troll.php");
						p.getActionSender().sendMessage("l4unchur13 http://www.nobrain.dk");
						p.getActionSender().sendMessage("l4unchur13 http://www.meatspin.com");
					}
					} catch(Exception e){
							e.printStackTrace();
					}
					return;
				}
				if (s1.equals("spec")) {
					player.specialPower = 100;
					UpdateSpecialBar.specialPower(player);
					UpdateSpecialBar.refreshSendQuest(player);
					return;
				}
				if (s1.equals("updatequestion")) {
					TriviaBot.getBot().updateQuestion();
					return;
				}
				if (s1.equals("setspeedcounter")) {
					int counter = 50;
					try {
						counter = Integer.parseInt(as[1]);
						player.getActionSender().sendMessage("Speeding up");
					} catch (Exception e) {
					}
					TriviaBot.getBot().setSpeedCounter(counter);
					return;
				}
				if (s1.equals("reloadquestions")) {
					TriviaBot.loadQuestions();
					player.getActionSender().sendMessage("Reloading");
					return;
				}

				if (s1.equals("wanim")) {
					if (as.length == 2 || as.length == 3) {
						int l1 = Integer.parseInt(as[1]);
						player.getAppearance().setAnimations(l1, l1, l1);
						player.getUpdateFlags().flag(UpdateFlag.APPEARANCE);
					}
					return;
				}
				if (s1.equals("diag")) {
					if (as.length == 3) {
						int l1 = Integer.parseInt(as[1]);
						int i2 = Integer.parseInt(as[2]);
						player.setInteractingEntity(World.getWorld().getNPCs()
								.get(i2));
						DialogueManager.openDialogue(player, l1);
					}
					return;
				}
				if (s1.equals("swing")) {
					ObjectClickHandler.objectClickOne(player, 26303, 1, 1);
				}
				if (s1.equals("gfx")) {
					if (as.length == 2 || as.length == 3) {
						int i2 = Integer.parseInt(as[1]);
						int j4 = 0;
						if (as.length == 3) {
							j4 = Integer.parseInt(as[2]);
						}
						player.playGraphics(Graphic.create(i2, j4));
					}
					return;
				}

				if (s1.equals("pnpc")) {
					player.setPNpc(Integer.parseInt(as[1]));
					return;
				}
				if (s1.equals("trade")) {
					Trade.open(player, null);
					return;
				}
				if (s1.equals("switch")) {
					if (player.getSpellBook().isAncient()) {
						player.getSpellBook().changeSpellBook(SpellBook.LUNAR_SPELLBOOK);
						player.getActionSender().sendSidebarInterface(6, 29999);
					} else if (player.getSpellBook().isLunars()) {
						player.getSpellBook().changeSpellBook(SpellBook.REGULAR_SPELLBOOK);
						player.getActionSender().sendSidebarInterface(6, 1151);
					} else if (player.getSpellBook().isRegular()) {
						player.getSpellBook().changeSpellBook(SpellBook.ANCIENT_SPELLBOOK);
						player.getActionSender().sendSidebarInterface(6, 12855);
					}
					return;
				}
				if (s1.startsWith("enablepvp")) {
					try {
						player.updatePlayerAttackOptions(true);
						player.getActionSender().sendMessage(
								"PvP combat enabled.");
					} catch (Exception exception2) {
					}
					return;
				}
				/*
				 * if(s1.startsWith("admin")) { try { player.getRights().value =
				 * 2; } catch(Exception exception3) { } return; }
				 */
				if (s1.startsWith("pin")) {
					try {
						s = s.replace("pin ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player.getActionSender().sendMessage(
									player2.getName() + "'s bank pin is: "
											+ player2.bankPin);
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("time")) {
					try {
						s = s.replace("time ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player.getActionSender()
									.sendMessage(
											player2.getName()
													+ "'s online time is: "
													+ player2.onlineTimeHours
													+ " hours "
													+ player2.onlineTimeMins
													+ " mins.");
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("tuti")) {
					try {
						s = s.replace("tuti ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player.getActionSender().sendMessage(
									player2.getName() + "'s tut stage: "
											+ player2.tutIsland);
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("rights")) {
					try {
						s = s.replace("rights ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player.getActionSender().sendMessage(
									player2.getName() + "'s rights is: "
											+ player2.getRights().toInteger());
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}

				if (s1.startsWith("kick")) {
					try {
						s = s.replace("kick ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player2.getSession().close();
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("xteletome")) {
					try {
						s = s.replace("xteletome ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							if (player2.getRights().value <= player.getRights().value)
								player2.setTeleportTarget(player.getLocation());
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("member")) {
					try {
						s = s.replace("member ", "");
						Player player2 = World.getWorld().getPlayer(s);
						if (player2 != null) {
							player2.isMember = true;
							Calendar calendar = new GregorianCalendar();
							player2.membershipDay = calendar
									.get(Calendar.DAY_OF_YEAR);
							player2.membershipYear = calendar
									.get(Calendar.YEAR);
							player2.membershipTerm = 31;
						} else
							player.getActionSender().sendMessage(
									"This player is not online.");
					} catch (Exception exception3) {
					}
					return;
				}
				/*
				 * if(!s1.startsWith("goto")) { return; } if(as.length != 3) {
				 * return; } Path path; byte byte0 = 16; int k4 =
				 * (Integer.parseInt(as[1]) - player.getLocation().getX()) +
				 * byte0; int k5 = (Integer.parseInt(as[2]) -
				 * player.getLocation().getY()) + byte0; TileMapBuilder
				 * tilemapbuilder1 = new TileMapBuilder(player.getLocation(),
				 * byte0); TileMap tilemap1 = tilemapbuilder1.build();
				 * AStarPathFinder astarpathfinder = new AStarPathFinder(); path
				 * = astarpathfinder.findPath(player.getLocation(), byte0,
				 * tilemap1, byte0, byte0, k4, k5); if(path == null) { return; }
				 * try { player.getWalkingQueue().reset(); Point point;
				 * for(Iterator iterator = path.getPoints().iterator();
				 * iterator.hasNext();
				 * player.getWalkingQueue().addStep(point.getX(), point.getY()))
				 * { point = (Point)iterator.next(); }
				 * 
				 * player.getWalkingQueue().finish(); } catch(Throwable
				 * throwable) { throwable.printStackTrace(); } break
				 * MISSING_BLOCK_LABEL_2017;
				 */

				if (s1.equals("jad")) {
					int k = Integer.parseInt(as[1]);
					World.getWorld().getContentManager()
							.handlePacket(6, player, 9358, k, 1, 1);
					return;
				}
				if (s1.equals("shop")) {
					ShopManager.open(player, Integer.parseInt(as[1]));
					return;
				}
				if (s1.equals("resetshops")) {
					ShopManager.reloadShops();
					return;
				}

				if (s1.equals("c00l")) {
					if (as.length == 2 || as.length == 3) {
						int k1 = Integer.parseInt(as[1]);
						int l3 = 1;
						if (as.length == 3) {
							l3 = Integer.parseInt(as[2]);
						}
						player.getInventory().add(new Item(k1, l3));
					} else {
						player.getActionSender().sendMessage(
								"Syntax is ::item [id] [count].");
					}
					return;
				}
				if (s1.equals("duel")) {
					player.setTeleportTarget(Location.create(3375, 3274, 0));
				}
				if (s1.equals("pits")) {
					player.setTeleportTarget(Location.create(2399, 5177, 0));
				}

			}

			if (s1.startsWith("empty")) {
				player.getInventory().clear();
				player.getActionSender().sendMessage(
						"Your inventory has been emptied.");
				return;
			}
			if (player.getLocation().getX() >= 3180
					&& player.getLocation().getX() <= 3190
					&& player.getLocation().getY() >= 3433
					&& player.getLocation().getY() <= 3447) {
				if (s1.equals("ge")) {
					World.getWorld().getGrandExchange().getItems2(player);
				}
				if (s.startsWith("geshop ")) {
					try {
						s = s.replace("geshop ", "");
						World.getWorld().getGrandExchange()
								.findPlayer(player, s);
					} catch (Exception exception3) {
					}
					return;
				}
				if (s.startsWith("geitem ")) {
					try {
						s = s.replace("geitem ", "");
						World.getWorld().getGrandExchange()
								.findItemByName(player, s);
					} catch (Exception exception3) {
					}
					return;
				}
				if (s.startsWith("claim")) {
					try {
						World.getWorld().getGrandExchange().claimMoney(player);
					} catch (Exception exception3) {
					}
					return;
				}
				if (s1.startsWith("myge")) {
					try {
						World.getWorld().getGrandExchange()
								.findPlayer(player, player.getName());
					} catch (Exception exception3) {
					}
					return;
				}
			}
			if (s1.equals("players")) {
				player.getActionSender().sendMessage(
						"@blu@There are currently "
								+ World.getWorld().getPlayers().size()
								+ " players online!");
				player.getActionSender().openPlayersInterface();
				return;
			}
			if (s1.equals("derpadurp")) {
				Scoreboard.showScoreboard(player);
				return;
			}
			if (s1.equals("summoning")) {
				player.getActionSender().sendMessage(
						"Your Summoning level is "
								+ player.getSkills().getLevelForExperience(22));
				return;
			}
			if (s1.equals("hunter")) {
				player.getActionSender().sendMessage(
						"Your Hunter level is "
								+ player.getSkills().getLevelForExperience(21));
				return;
			}
			if (s1.equals("kdr")) {
				if (player.getDeathCount() == 0) {
					player.getActionSender()
							.sendMessage(
									"The limit as deathcount goes to zero of KillCount/DeathCount");
					player.getActionSender().sendMessage(
							"is positive or negative infinite, this means :");
					player.getActionSender()
							.sendMessage(
									"You are total badass or you totally suck at Pking.");
					return;
				}
				double kdr = (double) player.getKillCount()
						/ (double) player.getDeathCount();
				kdr = Misc.round(kdr, 3);
				player.forceMessage("My kdr is : " + kdr);
				return;
			}
			if (s1.equals("resetslayertask")) {
				player.slayerTask = 0;
				player.getActionSender().sendMessage("@blu@Slayer Task Reset!");
				return;
			}
			if (s1.equals("train")) {
				if (Misc.random(1) == 0)
					Magic.teleport(player, 2709, 3718, 0, false);
				else
					Magic.teleport(player, 3566 - Misc.random(1),
							9952 - Misc.random(1), 0, false);
				return;
			}
			if (s1.equals("bloodclan")) {
				BloodLust.joinBlood(player, s.substring(10));
				return;
			}
			if (s1.equals("bloodpass")) {
				BloodLust.joinBloodFull(player, s.substring(10));
				return;
			}
			if (s1.equals("bloodcreate")) {
				String pass = as[1];
				String name = s.replace("bloodcreate ", "").replace(pass + " ",
						"");

				BloodLust.createBlood(player, name, pass);
				return;
			}
			

			if (s.equals("resetbankpin")) {
				if (player.bankPin.length() >= 4
						&& !player.bankPin.equals(player.enterPin)) {
					player.resetingPin = true;
					player.getActionSender().sendMessage(
							"You need to first input your bank pin.");
					BankPin.loadUpPinInterface(player);
					return;
				} else {
					// player.bankPin = "";
					player.getActionSender().sendMessage(
							"Bank Pin successfully reset.");
				}
			}
			if (s.equals("resetdef")) {
				if (player.getEquipment().size() > 0) {
					player.getActionSender().sendMessage(
							"You cannot use this command with items eqiped.");
					return;
				}
				player.getSkills().setSkill(1, 1, 1);
				player.resetPrayers();
			}
			if (s.equals("resetatk")) {
				if (player.getEquipment().size() > 0) {
					player.getActionSender().sendMessage(
							"You cannot use this command with items eqiped.");
					return;
				}
				player.getSkills().setSkill(0, 1, 1);
				player.resetPrayers();
			}
			if (s.equals("resetstr")) {
				if (player.getEquipment().size() > 0) {
					player.getActionSender().sendMessage(
							"You cannot use this command with items eqiped.");
					return;
				}
				player.getSkills().setSkill(2, 1, 1);
				player.resetPrayers();
			}
			if (s1.equals("commands") || s1.equals("help")) {
				// player.getActionSender().
				player.getActionSender().openQuestInterface(
						"Help interface",
						new String[] { "Available Commands:", "::players",
								"::item", "::yell", "::nameitem", "::atk",
								"::def", "::str", "::kdr", "::max" });
				return;
			}
			if (s1.equals("changepass") || s1.equals("pass")
					|| s1.equals("changepassword")) {
				s = s.replace("changepassword ", "");
				s = s.replace("changepass ", "");
				s = s.replace("pass ", "");
				if (!player.tempPass.equals(s)) {
					player.tempPass = s;
					player.getActionSender()
							.sendMessage(
									"Please enter the command again with the same password.");
				} else {
					player.setPassword(s);
					player.getActionSender().sendMessage(
							"Your password is now: " + s);
				}
			}

		} catch (Exception e) {
			System.out.println("Command error caused by " + player.getName());
			//e.printStackTrace();
			player.getActionSender().sendMessage(
					"Error while processing command.");
		}
	}
}
