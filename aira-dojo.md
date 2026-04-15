## walk through of aira-dojo framwork


### Agentbox workflow

+ 这里比较tricky的一个地方是 src/dojo/config_dataclasses/launcher/slurm.py 里面这个类用到了两次
  ```
  @dataclass
  class SlurmConfig(LauncherConfig):
      job_name: str = field(
          default=SI("${id}-${task.name}"),
          metadata={
              "help": "Name for the slurm job.",
              "exclude_from_hash": True,
          },
      )
  
      account: str = field(
          default="maui",
          metadata={
              "help": "Slurm account name",
              "exclude_from_hash": True,
          },
      )
      qos: str = field(
          default="maui_high",
          metadata={
              "help": "Quality of service level",
              "exclude_from_hash": True,
          },
      )
      nodes: int = field(
          default=1,
          metadata={
              "help": "Number of nodes to allocate",
          },
      )
      ntasks_per_node: int = field(
          default=1,
          metadata={
              "help": "Number of tasks per node",
              "exclude_from_hash": True,
          },
      )
      gpus_per_node: int = field(
          default=1,
          metadata={
              "help": "Number of GPUs per node",
          },
      )
      cpus_per_task: int = field(
          default=24,
          metadata={
              "help": "Number of CPU cores per task",
              "exclude_from_hash": True,
          },
      )
      partition: str | None = field(
          default=None,
          metadata={
              "help": "Slurm partition to use",
              "exclude_from_hash": True,
          },
      )
      time: int = field(
          default=1440,
          metadata={
              "help": "Maximum job runtime IN MINUTES",
              "exclude_from_hash": True,
          },
      )
      mem: str = field(
          default="200G",
          metadata={
              "help": "Memory allocation per node",
              "exclude_from_hash": True,
          },
      )
      array_parallelism: int | None = field(
          default=None,
          metadata={
              "help": "Maximum number of parallel jobs to run.",
              "exclude_from_hash": True,
          },
      )
  
      #################
      # Matrix autodeployment args
      #################
      auto_deploy_clients: bool = field(
          default=False,
          metadata={
              "help": "Whether to automatically deploy the models before launching the sweep.",
              "exclude_from_hash": True,
              "exclude_from_executor": True,
          },
      )
      deployment_account: str | None = field(
          default=None,
          metadata={
              "help": "Account name for the deployment.",
              "exclude_from_hash": True,
              "exclude_from_executor": True,
          },
      )
      deployment_qos: str | None = field(
          default=None,
          metadata={
              "help": "Quality of service level",
              "exclude_from_hash": True,
              "exclude_from_executor": True,
          },
      )
  
      def validate(self) -> None:
          super().validate()
  
          assert isinstance(self.time, int), "The time field should be an integer representing minutes."
  
          if self.auto_deploy_clients:
              assert self.deployment_account is not None, (
                  "If auto_deploy_clients is True, deployment_account must be provided."
              )
              assert self.deployment_qos is not None, "If auto_deploy_clients is True, deployment_qos must be provided.
  
  ```
+ 第一次用到是在main_runner_job_array.py 里面 因为src/dojo/configs/default_run.yaml 里面指定了要用src/dojo/configs/launcher/slurm.yaml 所以launcher 会被实例化为Class SlurmConfig 用于在main_runner_job_array.py的launch_jobs()函数里面负责构建submitit executor; 第二次用到是interpreter 指定为src/dojo/configs/interpreter/agentbox/slurm.yaml之后 使用对应的AgentboxInterpreterConfig class 其中的一个attr 名为slurm_args 也是一个SlurmConfig 它被用到是在src/dojo/core/interpreters/agentbox.py 里面的create_agentbox_service() 用的方式是：
  ```
  slurm_service_args = SlurmServiceAllocatorArguments(
            name=config.slurm_args.job_name + "agentbox_worker",
            timeout_min=config.slurm_args.time,
            cpus_per_node=config.slurm_args.cpus_per_task,
            gpus_per_node=config.slurm_args.gpus_per_node,
            account=config.slurm_args.deployment_account,
            qos=config.slurm_args.deployment_qos,
            mem=config.slurm_args.mem,
            partition=config.slurm_args.partition,
            use_arrays=bool(getattr(config.slurm_args, "use_arrays", False)),
            array_size=getattr(config.slurm_args, "array_size", None),
            array_max_parallelism=getattr(config.slurm_args, "array_parallelism", None),
            worker_logs_base_path=config.slurm_args.log_path,
        )
  ```
  注意这个地方用的是deployment_qos 和 deployment_account 虽然另外两个qos 和account 在这个config里面是不会被用到的
