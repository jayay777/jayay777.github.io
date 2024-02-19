---
title: Project BulletTime
layout: post
post-image: /assets/images/PBMainMenu.png
description: A card game with a twist. Slay the Spire meets XCom. Private project.
tags:
- Unity
- Single Player
- In Progress
---


<iframe width="560" height="315" src="https://www.youtube.com/embed/0D5SxUNGVfg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Project BulletTime is a private project I've been working on, bit by bit ever since I started Unity. To put it simply, it is a mix of Slay the Spire and XCom. The player uses cards to defeat enemies, and acquire more cards or upgrade them to go forward. The twist is that <b>time</b> is also a resource. Each card uses a certain amount of time unless it is used with a feature called cancel mode. It is a strategic real-time action game that is mixed up with Slay the Spire.
  
# Features I have implemented.

Aside from the 3D models, illustrations, and some of the music, everything has been done by myself. Including:

* The idea and the game system.

* The 50+ cards that have been implemented, and their upgrades

* UI & their animations

* The black market & workshop

* 10+ Enemy attacks, movements, and patterns

<div class="image-container">
  <img src="/assets/images/PBMap.png" alt="Image">
</div>

<div class="image-container">
  <img src="/assets/images/PBNyalpha.png" alt="Image">
</div>

<div class="image-container">
  <img src="/assets/images/PBMurdoc.png" alt="Image">
</div>



<div class="image-container">
  <img src="/assets/images/PBPlayerUML.PNG" alt="Image">
</div>


The following shows the source code for the PlayerController class above. It is one of the most crucial part of the game.
PlayerController acts as a sort of Facade pattern for the DeckManager so that it can simply hand over the the information about the card(<b>CardProperties</b>) and the position(Vector3) without needing to worry about how the player is going to have to move using Navcon or attack with FOV. DeckManager already has a lot to handle, so such pattern was definitely necessary.

The PlayerController code is shown below as an example. The Deckmanager only has to call methods such as StartAttack with just the CardProperties of the used cards, and the PlayerController handles the rest of the necessary components (such as camera, transform, attackFOV, etc). This makes it so that the DeckManager can focus on managing decks.

<details>
  <summary>PlayerController Code (Click to Open)</summary>

  <div markdown="1">
  ```C#
  public class PlayerController : MonoBehaviour
  {
      [SerializeField] private Material safeMaterial;
      [SerializeField] private Material inRangeMaterial;
      public CameraController cameraController;
      public AnimatorEventSounds animatorEventSounds;
      public RPGCharacterNavigationController navCon;
      [SerializeField] private GameObject attackFOVPrefab;
      private FOV attackFOV;
      private RPGCharacterController rpgCharacterController;
      private ParticleSystemRenderer particle;
      private bool decidingMovement = false;
      [HideInInspector] public DeckManager deckManager;

      void Start(){
          animatorEventSounds = GetComponentInChildren<AnimatorEventSounds>();
          attackFOV = Instantiate(attackFOVPrefab,Vector3.zero,Quaternion.identity).GetComponent<FOV>();
          attackFOV.playerLocation = transform;
          navCon = GetComponent<RPGCharacterNavigationController>();     
          rpgCharacterController = GetComponent<RPGCharacterController>();
          particle = GetComponentInChildren<ParticleSystemRenderer>();
          StartCoroutine(Intro());
          //StartCoroutine(cameraController.CameraStart());
      }
      private IEnumerator Intro(){
          yield return new WaitForSeconds(0.3f);
          rpgCharacterController.StartAction("SwitchWeapon", new SwitchWeaponContext("Unsheath", "None", "Back", 1, -1));
      }
      
      public void Deciding(CardProperties cardProperties, Vector3 hitPoint){
          hitPoint.y = transform.position.y;
          if (cardProperties.cardMoves){
              DecidingMovement(hitPoint, cardProperties.moveDistance, cardProperties.moveType);
          }
          if (cardProperties.cardAttacks){
              if (cardProperties.cardMoves && !AssetManager.i.nav.activeSelf){ // If the movement isn't allowed
                  attackFOV.Clear();
                  return;
              }
              DecidingAttack(hitPoint, cardProperties);
          }
      }
      /// <summary>
      /// Sets destination, movedistance, and set bool to starting drawing path
      /// </summary>
      ///<param name="dest">The destination point</param>
      ///<param name="MoveDistance">The move distance of the used card</param>
      ///<param name="moveType">0 for instant, 1 for default, 2 for linear, 3 for linear + fixed distance.</param>
      private void DecidingMovement(Vector3 dest, float MoveDistance, int moveType) {
          cameraController.FollowMouse(true);    
          decidingMovement = true;
          navCon.DecidingMovement(dest, MoveDistance,moveType);
      }

      private Vector3 lastDestination;
      private float lastAttackRange;
      private float lastAttackDegrees;
      private void DecidingAttack(Vector3 hitpoint, CardProperties cardProperties) {
          cameraController.FollowMouse(true);
          lastAttackRange = cardProperties.attackRange;
          switch (cardProperties.attackType){
              case 0: //Melee
                  lastDestination = hitpoint;
                  lastAttackDegrees = cardProperties.attackDegrees;            
                  attackFOV.ViewAttackMelee((hitpoint - transform.position).normalized,cardProperties);
                  break;
              case 1: //Ranged
                  lastDestination = hitpoint;
                  attackFOV.ViewAttackRanged((hitpoint - transform.position).normalized,cardProperties);
                  break;
              case 2: //Collider
                  break;
              case 3: //melee(reverse)
                  lastDestination = 2*transform.position - hitpoint;
                  lastAttackDegrees = cardProperties.attackDegrees;            
                  attackFOV.ViewAttackMelee((transform.position - hitpoint).normalized,cardProperties);
                  break;
              case 4: //ranged(reverse)
                  lastDestination = 2*transform.position - hitpoint;
                  attackFOV.ViewAttackRanged((transform.position - hitpoint).normalized,cardProperties);
                  break;
              default:
                  Debug.LogError("Wrong attack type");
                  break;
          } 
      }

      public void StopDecision() {
          cameraController.FollowMouse(false);
          decidingMovement = false;
          navCon.ClearPathRenderer();
          attackFOV.Clear();
      }
      public void ClearMovementAndAttack(){
          attackFOV.Clear();
          navCon.ClearPathRenderer();
          navCon.StopNavigating();
      }

      public void StartMoving(CardProperties cardProperties) {
          cameraController.FollowMouse(false);
          decidingMovement = false;
          StartCoroutine(MovementDelay(cardProperties));
          //NavCon.MeshNavToPoint(finalDestination);
      }
      
      private IEnumerator MovementDelay(CardProperties cardProperties){
          yield return new WaitForSeconds(cardProperties.moveDelay);
          bool rotation = true;
          if (cardProperties.animNum1 == -1) rpgCharacterController.Roll(1);
          else if(cardProperties.animNum1 == 0 && cardProperties.animNum3 == 0){
              rpgCharacterController.SetAnimatorTrigger(AnimatorTrigger.InstantMovementTrigger);
          }
          else if(cardProperties.animNum3 != 0){
              rotation = false;
              rpgCharacterController.Roll(cardProperties.animNum3);
          }
          navCon.StartMoving(cardProperties.moveSpeed,rotation);
      }

      public void StartAttack(CardProperties cardProperties){
          if(cardProperties.animNum1!=0) rpgCharacterController.StartAction("SwitchWeapon", new SwitchWeaponContext("Instant", "None", "Back", cardProperties.animNum1, -1));
          StartCoroutine(CheckAndAttackAnimation(cardProperties.animNum1, cardProperties.animNum2));    
          if (cardProperties.attackType == 1) {
              StartRangedAttack(cardProperties.damage, cardProperties.attackHitTime,cardProperties.debuffNumber,cardProperties.debuffAmount);
          }
          else{
              StartMeleeAttack(cardProperties.damage, cardProperties.attackHitTime,cardProperties.debuffNumber,cardProperties.debuffAmount);
          }
      }
      private IEnumerator CheckAndAttackAnimation(int animNum1,int animNum2){
          for(int i = 0; i < 5; i++){
              yield return new WaitForEndOfFrame();
              if (rpgCharacterController.CanStartAction("SwitchWeapon")){
                  if(animNum1 == 1){
                      rpgCharacterController.StartAction("Attack", new AttackContext("Attack", "None", animNum2));
                  }
                  else if(animNum2 > 1000){
                      rpgCharacterController.StartAction("Attack", new AttackContext("Kick", "None", animNum2%1000));
                  }
                  else{
                      rpgCharacterController.StartAction("Attack", new AttackContext("Attack","Right", animNum2));
                  }
                  yield break;
              }
              //Debug.Log(i + " frames");
          }
          Debug.LogError("Weapon Not Switched");
          rpgCharacterController.StartAction("Attack", new AttackContext("Attack", "None", animNum2));
      }

      public void StartMeleeAttack(int damage, float attackHitTime,int debuffNumber,float debuffAmount) {
          transform.rotation = Quaternion.LookRotation((lastDestination - transform.position).normalized);
          cameraController.FollowMouse(false);
          attackFOV.AttackMelee(lastAttackRange, lastAttackDegrees,
            (lastDestination - transform.position).normalized, damage, attackHitTime, debuffNumber,debuffAmount);
          //Debug.Log("dealt" + damage + "damage!");
      }
      public void StartRangedAttack(int damage, float attackHitTime,int debuffNumber,float debuffAmount){
          cameraController.FollowMouse(false);
          if (navCon.isNavigating) navCon.StopNavigating();
          transform.rotation = Quaternion.LookRotation((lastDestination - transform.position).normalized);
          attackFOV.AttackRanged(lastAttackRange,
            (lastDestination - transform.position).normalized, damage, attackHitTime,debuffNumber,debuffAmount);
          //Debug.Log("dealt" + damage + "damage!");
      }

      void Update(){
          if (decidingMovement){
              navCon.Navigate();
          }
      }

      public void OutlineOn(bool onoff){
          if (onoff) particle.material = inRangeMaterial;
          else{
              particle.material = safeMaterial;
          }
      }

      public void HitByEnemy(int damage){
          deckManager.TakeHit(damage);
      }

      public void Defeat(){
          StopDecision();
          ClearMovementAndAttack();
          rpgCharacterController.Death();
      }
  }
  ```
  </div>
</details>

